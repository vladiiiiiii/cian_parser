<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0f172a,45:2563eb,100:14b8a6&height=180&section=header&text=CIAN%20Real%20Estate%20Parser&fontColor=ffffff&fontSize=34&fontAlignY=38&desc=Scraper%20%2B%20Geocoding%20pipeline%20for%20ML-ready%20listing%20data&descAlignY=58&descSize=15&animation=fadeIn" alt="CIAN Parser banner" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776ab?style=flat-square&logo=python&logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/Playwright-2ead33?style=flat-square&logo=playwright&logoColor=white" alt="Playwright" />
  <img src="https://img.shields.io/badge/BeautifulSoup-white?style=flat-square&logo=python&logoColor=black" alt="BeautifulSoup" />
  <img src="https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=pandas&logoColor=white" alt="Pandas" />
  <img src="https://img.shields.io/badge/Geopy%20%2F%20Nominatim-4caf50?style=flat-square&logo=openstreetmap&logoColor=white" alt="Geopy" />
</p>

## About

A data collection pipeline that scrapes real estate listings from **cian.ru** (Chelyabinsk region), cleans and structures the data, geocodes addresses into coordinates, and outputs a dataset ready to be consumed by an ML pricing model and a backend service.

This was built as the data-collection stage for the [RealtorConnect](https://github.com/vladiiiiiii/Realtor-connect-backend) project — the resulting dataset feeds directly into the CatBoost price prediction model used in that app.

## How It Works

The pipeline went through a few iterations, kept in the notebook as a record of what actually worked:

1. **Plain `requests` + BeautifulSoup** — fast, but CIAN's anti-bot protection blocks a simple HTTP session fairly quickly.
2. **`cianparser` library** — tried as a ready-made solution with proxy rotation, but hit reliability limits.
3. **Playwright with headless Chromium** — the approach that actually works reliably. It renders the page like a real browser, which is necessary to get past CIAN's bot detection.

The final pipeline (Playwright-based) is the one used in production.

## Pipeline Steps

1. **Fetch** — load listing pages with Playwright (headless Chromium), configurable page limit per category (`MAX_PAGES_PER_CATEGORY`).
2. **Parse** — extract raw text blocks with BeautifulSoup, then pull out title, price, area, floor, rooms, and description with regex-based parsers.
3. **Clean** — normalize price (handles `₽` and `млн` formats), area, and floor into proper numeric types; drop rows with missing critical fields.
4. **Geocode** — resolve street addresses to latitude/longitude using `geopy` + Nominatim, with house-number extraction from the listing title as a fallback.
5. **Export** — save the cleaned dataset as timestamped CSV and pickle files, with columns matching the backend's `Apartment` data model (`Address`, `Rooms`, `Area`, `Price`, `Description`, `Floor`, `HousingType`, `Latitude`, `Longitude`).

## Extracted Fields

| Field | Description |
|---|---|
| `Address` | Cleaned street address |
| `Rooms` | Number of rooms |
| `Area` | Total area, m² |
| `Price` | Listing price, ₽ |
| `Floor` | Floor number |
| `HousingType` | Новостройка / Вторичное |
| `Description` | Listing description text |
| `Latitude` / `Longitude` | Geocoded coordinates |
| `PricePerSqm` | Derived: price per square meter |

## Tech Stack

- **Python** — pandas, requests, BeautifulSoup4, lxml
- **Playwright** (Chromium) — headless browser scraping to bypass anti-bot protection
- **geopy** (Nominatim) — address geocoding
- **nest_asyncio** — running async Playwright calls inside a notebook

## Getting Started

```bash
pip install pandas requests beautifulsoup4 lxml playwright geopy nest_asyncio
playwright install chromium
```

Run the notebook top to bottom; the final (Playwright-based) section produces the working dataset. Output files are saved under `data/2026/processed/`.

## Notes

- Scraping targets a specific region (`chelyabinsk.cian.ru`) and page limit is configurable — adjust `BASE_URL` and `MAX_PAGES_PER_CATEGORY` for other regions or larger runs.
- Respect CIAN's `robots.txt` and rate-limit requests; the pipeline already adds delays between requests to avoid overloading the source site.

## Related Repositories

- [RealtorConnect Backend](https://github.com/vladiiiiiii/Realtor-connect-backend) — consumes this dataset for the ML price estimation feature

<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:14b8a6,55:2563eb,100:0f172a&height=100&section=footer" alt="Footer" />
</p>
