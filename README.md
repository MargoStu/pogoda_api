# pogoda_api
pogoda_api

import requests
import json
from datetime import datetime, timedelta

LATITUDE = 52.2297  
LONGITUDE = 21.0122
CACHE_FILE = "wyniki_pogody.json"

def wczytaj_cache():
    try:
        with open(CACHE_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError:
        return {}

def zapisz_cache(dane):
    with open(CACHE_FILE, "w", encoding="utf-8") as f:
        json.dump(dane, f, indent=4, ensure_ascii=False)

def pobierz_pogode(data):
    url = (
        f"https://api.open-meteo.com/v1/forecast?"
        f"latitude={LATITUDE}&longitude={LONGITUDE}"
        f"&hourly=rain&daily=rain_sum"
        f"&timezone=Europe%2FLondon"
        f"&start_date={data}&end_date={data}"
    )

    response = requests.get(url)
    if response.status_code != 200:
        return None

    dane = response.json()
    return dane.get("daily", {}).get("rain_sum", [None])[0]

def ocen_opady(wartosc):
    if wartosc is None or wartosc < 0:
        return "Nie wiem"
    elif wartosc == 0.0:
        return "Nie będzie padać"
    else:
        return "Będzie padać"

def main():
    data_input = input("Podaj datę (YYYY-mm-dd) [Enter = jutro]: ").strip()

    if not data_input:
        data = (datetime.now() + timedelta(days=1)).strftime("%Y-%m-%d")
    else:
        try:
            datetime.strptime(data_input, "%Y-%m-%d")
            data = data_input
        except ValueError:
            print("Niepoprawny format daty! Użyj formatu YYYY-mm-dd.")
            return

    cache = wczytaj_cache()

    if data in cache:
        wynik = cache[data]
        print(f"[Z pliku] {data}: {wynik}")
    else:
        wartosc_opadow = pobierz_pogode(data)
        wynik = ocen_opady(wartosc_opadow)
        cache[data] = wynik
        zapisz_cache(cache)
        print(f"[Z API] {data}: {wynik}")

if __name__ == "__main__":
    main()
