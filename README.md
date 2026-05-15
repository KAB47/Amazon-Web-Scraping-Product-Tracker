# Amazon Price Tracker — Web Scraping Project

An automated Python web scraping project that tracks the price of a Dell XPS 15 on Amazon UK over time, logs data to a CSV, and runs on a daily schedule.

---

## TL;DR

Scrapes the title and price of a Dell XPS 15 (Or any product) listing on Amazon daily using BeautifulSoup and Requests, appends each reading to a CSV with a timestamp, and loops automatically every 24 hours to build a price history dataset.

---

## Main Features / Outcomes

- Scrapes live product title and price from an Amazon UK product page
- Cleans and strips raw HTML text for usable data
- Logs each scrape as a timestamped row in a CSV (`ProjectDataScraperDataset.csv`)
- Appends new data on each run without overwriting existing records
- Runs on an automated 24-hour loop via `time.sleep(86400)`

---

## Background

The goal of this project is to monitor product price fluctuations on Amazon UK over time without checking manually. By scraping and recording the price daily, you can spot patterns, drops, or spikes in pricing — useful for finding the best time to buy.

The product tracked: **Dell XPS 15 9510** (Renewed) — Amazon UK listing.

---

## How It Works

### Step 1 — Connect & Scrape
Sends an HTTP GET request to the Amazon product page using a browser-like `User-Agent` header to avoid being blocked. Parses the response HTML with BeautifulSoup to extract the product title and price.

```python
page = requests.get(URL, headers=headers)
soup = BeautifulSoup(page.content, "html.parser")
title = soup.find(id='productTitle').get_text().strip()
price = soup.find(class_='a-price-whole').get_text().strip()
```

### Step 2 — Clean the Data
Strips excess whitespace and newline characters from the raw scraped text to produce clean, readable values.

### Step 3 — Log to CSV
Writes the title, price, and today's date to a CSV file. On the first run, the header row is written. On subsequent runs, data is appended so no records are lost.

```python
header = ['Title', 'Price (GBP)', 'Date (YYYY/MM/DD)']
data   = [title, price, today]

with open('ProjectDataScraperDataset.csv', 'a+', newline='', encoding='UTF8') as f:
    writer = csv.writer(f)
    writer.writerow(data)
```

### Step 4 — Automate
The `check_price()` function wraps the full scrape-and-log flow. A `while True` loop calls it every 86,400 seconds (24 hours), building the dataset automatically over time.

```python
while True:
    check_price()
    time.sleep(86400)
```

---

## Known Issues & Fixes

| Issue | Description | Fix |
|---|---|---|
| `PermissionError` on CSV write | File was open in another application (e.g. Excel) while the script tried to write | Close the file before running the script |
| Price includes stray whitespace/newlines | Raw HTML text contains formatting characters | Use `.strip()` and slice the string as needed |
| `datetime.date.today` not called | Missing `()` in the `check_price()` function — returns the method, not the date | Change to `datetime.date.today()` |
| Hardcoded CSV path | `pd.read_csv()` uses an absolute local path, breaking on other machines | Use a relative path or `os.path` for portability |

---

## Tools & Stack

- **Python 3** — core scripting language
- **BeautifulSoup4** — HTML parsing
- **Requests** — HTTP page fetching
- **csv** — writing and appending to CSV files
- **pandas** — reading and inspecting the CSV dataset
- **datetime** — timestamping each scrape
- **time** — scheduling the 24-hour loop

---

## Prerequisites

Python 3.x — [python.org/downloads](https://python.org/downloads)

Install required libraries:

```bash
pip install beautifulsoup4 requests pandas
```

---

## Running the Project

1. Clone or download the project folder
2. Open the notebook (`Amazon_Web_Scraping_Project.ipynb`) in Jupyter
3. Update the `URL` variable if the Amazon listing changes
4. Update the `User-Agent` header with your own — visit [httpbin.org/get](https://httpbin.org/get) in your browser to find it
5. Run all cells in order
6. The script will begin logging to `ProjectDataScraperDataset.csv` in the same directory

> **Note:** Make sure the CSV file is not open in Excel or another application when the script runs, or you will get a `PermissionError`.

---

## Project Structure

```
├── Amazon_Web_Scraping_Project.ipynb   # Main notebook with all scraping logic
├── ProjectDataScraperDataset.csv       # Output dataset (auto-generated on first run)
└── README.md
```

---

## Potential Extensions

- Add an email alert via `smtplib` to notify when the price drops below a set threshold
- Scrape multiple products in a single run and log to separate CSV columns
- Visualise price history over time using `matplotlib` or `pandas` plotting
- Schedule via Windows Task Scheduler or cron instead of a blocking `while` loop
- Store data in a SQLite database instead of CSV for easier querying
