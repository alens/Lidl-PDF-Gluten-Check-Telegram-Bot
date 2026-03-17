
## 🛠️ 1. The Sanitized Script (`lidl_scanner.py`)

```python
import requests
import re
import fitz  # PyMuPDF
import os
import sys

# --- CONFIGURATION VIA ENVIRONMENT VARIABLES ---
# It is best practice not to hardcode secrets in public repos.
TOKEN = os.getenv("TELEGRAM_TOKEN")
CHAT_ID = os.getenv("TELEGRAM_CHAT_ID")

# Hidden API for Lidl Greece flyer discovery
DISCOVERY_API = "https://mobile.lidl.de/Mobile-Server/service/7/containerService/GR/flyer/el"
KEYWORDS = ["gluten", "γλουτένη", "χωρίς γλουτένη", "χωρις γλουτενη", "gluten-free"]
LAST_SCANNED_FILE = "last_scanned_id.txt"

def send_telegram(text):
    if not TOKEN or not CHAT_ID:
        print("Telegram credentials missing. Skipping notification.")
        return
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    payload = {"chat_id": CHAT_ID, "text": text, "parse_mode": "Markdown"}
    try:
        requests.post(url, json=payload, timeout=10)
    except Exception as e:
        print(f"Telegram Error: {e}")

def get_latest_pdf_url():
    headers = {'User-Agent': 'LidlPlus/1.0'}
    try:
        response = requests.get(DISCOVERY_API, headers=headers, timeout=20)
        # Search for PDF URLs in the API response (XML/JSON compatible regex)
        pdf_urls = re.findall(r'<pdfUrl>(.*?)</pdfUrl>', response.text)
        if not pdf_urls:
            pdf_urls = re.findall(r'"pdfUrl"\s*:\s*"(.*?)"', response.text)

        for url in pdf_urls:
            clean_url = url.replace('&amp;', '&')
            # Focus on the main Food/Weekly flyer
            if "Food" in clean_url or "Phylladio" in clean_url:
                return clean_url
    except Exception as e:
        print(f"Discovery Error: {e}")
    return None

def main():
    pdf_url = get_latest_pdf_url()
    
    if not pdf_url:
        print("No flyer found on API.")
        return

    # Once-only check using a local memory file
    if os.path.exists(LAST_SCANNED_FILE):
        with open(LAST_SCANNED_FILE, "r") as f:
            if f.read().strip() == pdf_url:
                print("Flyer already scanned. Exiting.")
                return

    print(f"New Flyer detected: {pdf_url}")
    
    try:
        r = requests.get(pdf_url, timeout=120)
        doc = fitz.open(stream=r.content, filetype="pdf")
        
        found_pages = []
        for i in range(len(doc)):
            text = doc.load_page(i).get_text().lower()
            matches = [k for k in KEYWORDS if k in text]
            if matches:
                found_pages.append(f"• **Page {i+1}**: {', '.join(set(matches))}")

        if found_pages:
            summary = "\n".join(found_pages)
            msg = (f"🍞 **Lidl Greece Gluten Alert**\n"
                   f"A new flyer was detected with keyword matches:\n\n"
                   f"{summary}\n\n"
                   f"🔗 [View Full PDF]({pdf_url})")
            send_telegram(msg)
            print("Notification sent.")
        else:
            print("No keywords found.")

        # Update the memory file
        with open(LAST_SCANNED_FILE, "w") as f:
            f.write(pdf_url)
            
    except Exception as e:
        print(f"Error during processing: {e}")

if __name__ == "__main__":
    main()
```

---

## 📖 2. The `README.md`

```markdown
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
```

---

### 💡 Quick Tips for GitHub:
* **.gitignore**: Create a file named `.gitignore` and add `last_scanned_id.txt` and `__pycache__/` to it. You don't want your "memory" files tracking on GitHub.
* **License**: Since you are making this public, consider adding an **MIT License** file so others know they can use and improve it.
