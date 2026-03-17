# Lidl Greece Flyer Allergen Scanner 🇬🇷🍞

An automated Python tool designed to monitor Lidl Hellas weekly flyers for specific dietary keywords (like "gluten-free"). It uses Lidl's internal discovery API to find new PDFs, scans them for keywords using PyMuPDF, and notifies you via Telegram.

## Features
* **Auto-Discovery**: Queries the Lidl Mobile API to find new flyers automatically.
* **Smart Scanning**: Uses `PyMuPDF` for fast, local text extraction from high-res PDFs.
* **Anti-Spam**: Remembers the last scanned flyer UUID so you only get notified once per week.
* **Headless**: Designed to run on a Linux CLI/VPS via Cron.

## Prerequisites
* Python 3.x
* `requests`
* `pymupdf` (fitz)

Install dependencies:
```bash
pip install requests pymupdf
```

## Setup & Configuration

This script uses **Environment Variables** for security. You must set these on your system before running:

1. **Get a Telegram Bot Token**: Talk to [@BotFather](https://t.me/botfather).
2. **Get your Chat ID**: Talk to [@IDBot](https://t.me/myidbot).

### Setting Variables (Linux)
Add these to your `~/.bashrc` or export them in your terminal:
```bash
export TELEGRAM_TOKEN="your_bot_token_here"
export TELEGRAM_CHAT_ID="your_chat_id_here"
```

## Usage
Run the script manually:
```bash
python3 lidl_scanner.py
```

### Automation (Cron)
To check for new flyers every day at 10:00 AM, add this to your `crontab -e`:
```cron
0 10 * * * TELEGRAM_TOKEN="xxx" TELEGRAM_CHAT_ID="yyy" /usr/bin/python3 /path/to/lidl_scanner.py
```

## Disclaimer
This project is for educational and personal use only. It is not affiliated with, authorized, maintained, sponsored, or endorsed by Lidl. Use responsibly and respect the host's `robots.txt` and rate limits.


