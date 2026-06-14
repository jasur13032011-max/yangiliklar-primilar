# yangiliklar-primilar
import requests
import re
import csv
import json

from dataclasses import dataclass, asdict


# ==========================
# DATACLASS
# ==========================

@dataclass
class Yangilik:
    id: int
    sarlavha: str
    matn: str
    sana: str
    teglar: list


# ==========================
# API DAN OLISH
# ==========================

def get_news():
    url = "https://jsonplaceholder.typicode.com/posts"

    response = requests.get(
        url,
        timeout=10
    )

    response.raise_for_status()

    return response.json()


# ==========================
# MATNNI TOZALASH
# ==========================

def clean_text(text):
    # HTML teglarni o'chirish
    text = re.sub(
        r"<[^>]+>",
        "",
        text
    )

    # Ko'p bo'shliqlarni bitta qilish
    text = re.sub(
        r"\s+",
        " ",
        text
    )

    return text.strip()


# ==========================
# SO'ZLARNI SANASH
# ==========================

def count_words(text):
    return len(
        re.findall(
            r"\b\w+\b",
            text
        )
    )


# ==========================
# OBYEKTLARGA AYLANTIRISH
# ==========================

def create_news_objects(raw_news):

    result = []

    for item in raw_news:

        news = Yangilik(
            id=item["id"],
            sarlavha=clean_text(item["title"]),
            matn=clean_text(item["body"]),
            sana="2026-06-14",
            teglar=["api", "news"]
        )

        result.append(news)

    return result


# ==========================
# CSV GA YOZISH
# ==========================

def save_csv(news_list):

    with open(
        "news.csv",
        "w",
        newline="",
        encoding="utf-8-sig"
    ) as file:

        writer = csv.DictWriter(
            file,
            fieldnames=[
                "id",
                "sarlavha",
                "matn",
                "sana",
                "teglar"
            ]
        )

        writer.writeheader()

        for news in news_list:
            writer.writerow(asdict(news))


# ==========================
# JSON GA YOZISH
# ==========================

def save_json(news_list):

    with open(
        "news.json",
        "w",
        encoding="utf-8"
    ) as file:

        json.dump(
            [asdict(n) for n in news_list],
            file,
            ensure_ascii=False,
            indent=4
        )


# ==========================
# STATISTIKA
# ==========================

def calculate_stats(news_list):

    total_news = len(news_list)

    total_words = sum(
        count_words(n.matn)
        for n in news_list
    )

    longest_news = max(
        news_list,
        key=lambda n: count_words(n.matn)
    )

    print("\nSTATISTIKA")
    print("-" * 30)

    print("Jami yangilik:", total_news)
    print("Jami so'z:", total_words)

    print(
        "Eng uzun yangilik ID:",
        longest_news.id
    )

    print(
        "So'zlar soni:",
        count_words(longest_news.matn)
    )


# ==========================
# PIPELINE
# ==========================

def run_pipeline():

    print("Yangiliklar yuklanmoqda...")

    raw_news = get_news()

    news_objects = create_news_objects(
        raw_news
    )

    save_csv(news_objects)

    save_json(news_objects)

    calculate_stats(news_objects)

    print("\nTayyor!")
    print("CSV va JSON saqlandi")


# ==========================
# START
# ==========================

if __name__ == "__main__":
    run_pipeline()
