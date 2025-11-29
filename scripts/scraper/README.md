# News Scraper for Hong Kong Fire Documentary

A parallel web scraper that automatically archives news articles from markdown link collections. Designed to run 24/7 on a Linux machine, syncing with upstream repositories and creating pull requests.

## Features

- **Parallel scraping** - Scrapes multiple domains simultaneously (up to 5 at once)
- **Duplicate detection** - Tracks scraped URLs to avoid re-scraping
- **Auto-sync** - Syncs with upstream repo every 10 minutes
- **Auto-PR** - Creates pull requests to upstream every hour
- **Rate limiting** - Configurable delays per domain to be respectful
- **Logging** - All activity logged to `logs/scraper.log`

## Quick Start

### Option 1: Automated Setup (Ubuntu/Debian)

```bash
cd Hong-Kong-Fire-Documentary
chmod +x scripts/scraper/setup_ubuntu.sh
./scripts/scraper/setup_ubuntu.sh
```

The script will:

1. Install system dependencies
2. Create Python virtual environment
3. Install Python packages and Playwright
4. Prompt for your GitHub token and fork repo
5. Optionally install as a systemd service

### Option 2: Manual Setup

```bash
# 1. Install dependencies
pip install -r scripts/scraper/requirements.txt
playwright install chromium
playwright install-deps chromium  # Linux only

# 2. Set environment variables
export GITHUB_TOKEN="your_github_personal_access_token"
export FORK_REPO="your-username/Hong-Kong-Fire-Documentary"

# 3. Run the scraper
python scripts/scraper/scraper.py --dry-run  # Test first
python scripts/scraper/scraper.py            # Actually scrape
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GITHUB_TOKEN` | Yes (for daemon) | GitHub Personal Access Token with `contents:write` and `pull_requests:write` |
| `FORK_REPO` | Yes (for daemon) | Your fork's repo path, e.g., `username/Hong-Kong-Fire-Documentary` |
| `UPSTREAM_REPO` | No | Upstream repo (default: `Hong-Kong-Emergency-Coordination-Hub/Hong-Kong-Fire-Documentary`) |
| `PR_BRANCH` | No | Branch name for PRs (default: `scraper-updates`) |
| `MAIN_BRANCH` | No | Main branch name (default: `main`) |

## Scripts

### `scraper.py` - Main Scraper

Extracts URLs from markdown files and scrapes them in parallel.

```bash
# List available news sources
python scraper.py --list-sources

# Dry run - show what would be scraped
python scraper.py --dry-run

# Scrape specific source only
python scraper.py --source BBC

# Limit number of URLs
python scraper.py --limit 10

# Verbose output
python scraper.py -v

# Combine options
python scraper.py --source HK01 --limit 5 --dry-run -v
```

### `daemon.py` - 24/7 Daemon Service

Runs continuously, syncing and scraping on schedule.

```bash
# Run daemon (runs forever)
python daemon.py

# Run single cycle and exit (for testing)
python daemon.py --once
```

**Schedule:**

- Every 10 minutes: Sync with upstream, detect new URLs, scrape them
- Every 60 minutes: Close old PR (if any), create new PR to upstream

## Configuration

### `config.yml` - Scraper Settings

```yaml
rate_limit:
  delay_seconds: 3      # Default delay between requests
  max_retries: 3        # Retry attempts on failure
  timeout_seconds: 60   # Page load timeout

user_agent: "Mozilla/5.0 ..."  # Browser user agent

# Per-site overrides
sites:
  bbc.co.uk:
    delay_seconds: 5    # BBC needs longer delays
  scmp.com:
    delay_seconds: 5
```

### `scraped_urls.json` - URL Registry

Automatically maintained. Tracks all scraped URLs to prevent duplicates.

```json
{
  "scraped_urls": {
    "https://example.com/article": {
      "title": "Article Title",
      "source": "BBC",
      "scraped_at": "2025-11-29T00:00:00",
      "archive_path": "content/news/BBC/archive/article-title"
    }
  },
  "last_updated": "2025-11-29T00:00:00"
}
```

## Output Structure

Scraped articles are saved in `archive/` subdirectories:

```
content/news/
├── BBC/
│   ├── readme.md           # Original with links
│   └── archive/
│       └── article-title/
│           ├── index.html      # Full HTML
│           └── metadata.json   # URL, title, timestamp
├── HK01/
│   ├── README.MD
│   └── archive/
│       └── ...
```

## Running as a Service (systemd)

For 24/7 operation on Ubuntu/Debian:

```bash
# 1. Create environment file
cat > ~/.scraper_env << EOF
GITHUB_TOKEN=your_token_here
FORK_REPO=your-username/Hong-Kong-Fire-Documentary
EOF
chmod 600 ~/.scraper_env

# 2. Edit service file (update paths and username)
sudo cp scripts/scraper/scraper.service /etc/systemd/system/news-scraper.service
sudo nano /etc/systemd/system/news-scraper.service

# 3. Enable and start
sudo systemctl daemon-reload
sudo systemctl enable news-scraper
sudo systemctl start news-scraper

# 4. Check status
sudo systemctl status news-scraper
journalctl -u news-scraper -f
```

## GitHub Token Setup

1. Go to <https://github.com/settings/tokens?type=beta>
2. Click "Generate new token"
3. Select your fork repository
4. Grant permissions:
   - **Contents**: Read and Write
   - **Pull requests**: Read and Write
5. Copy the token and save it securely

## Troubleshooting

### "GITHUB_TOKEN environment variable not set"

```bash
export GITHUB_TOKEN="your_token_here"
# Or source your env file:
source ~/.scraper_env
```

### "FORK_REPO environment variable not set"

```bash
export FORK_REPO="your-username/Hong-Kong-Fire-Documentary"
```

### Timeout errors on certain sites

Some sites (BBC, SCMP) have anti-bot protection. The scraper will retry 3 times automatically. If still failing:

- Increase `timeout_seconds` in `config.yml`
- Increase `delay_seconds` for that specific site
- Some sites may not be scrapable (paywall, Cloudflare)

### Service won't start

```bash
# Check logs for errors
journalctl -u news-scraper -n 50

# Common issues:
# - Wrong paths in service file
# - Missing environment file
# - Python venv not activated
```

### Service starts before WiFi is connected

The service file uses `network-online.target` to wait for network connectivity. If you're still experiencing issues:

```bash
# Check if network-online.target is working
systemctl status network-online.target

# If using WiFi, ensure NetworkManager is configured:
sudo systemctl enable NetworkManager-wait-online.service

# Reload and restart the service
sudo systemctl daemon-reload
sudo systemctl restart news-scraper
```

## File Reference

| File | Purpose |
|------|---------|
| `scraper.py` | Main scraper with parallel processing |
| `daemon.py` | 24/7 daemon service |
| `config.yml` | Rate limits and site-specific settings |
| `requirements.txt` | Python dependencies |
| `scraped_urls.json` | Registry of scraped URLs (auto-generated) |
| `setup_ubuntu.sh` | Automated setup script for Ubuntu |
| `scraper.service` | systemd service template |

## License

Part of the Hong Kong Fire Documentary project. See main repository for license.
