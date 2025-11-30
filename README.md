## 九、技術資源 (Technical Resources)

本存庫包含自動化工具及 CI/CD 流程，供開發者參考：

- **`.github/workflows/`**：GitHub Actions 自動化流程
  - `publish.yml`：自動構建並發布網站至 GitHub Pages
  - `lint.yml`：自動檢查 Markdown 格式及代碼風格 (Linting)
  - `auto-review.yml`：自動通知專案維護者要求審視 Pull Request
- **`scripts/scraper/`**：新聞自動爬蟲 (News Scraper)
  - 用於定時抓取各大媒體的最新報道
  - 包含 `scraper.py` (主程式) 及 `scraped_urls.json` (已抓取紀錄)
- **`scripts/archive_news.py`**：新聞連結自動備份工具
  - **功能**：自動掃描 Markdown 檔案中的連結，將其備份至 Internet Archive (Wayback Machine)，並在原連結旁加上備份連結。
  - **使用方法**：
    ```bash
    # 安裝依賴
    pip install -r scripts/requirements.txt
    
    # 掃描並備份 content/news 目錄下的所有 Markdown 檔案
    python3 scripts/archive_news.py content/news
    
    # 掃描並備份單個檔案
    python3 scripts/archive_news.py content/news/rthk/README.md
    ```
  - **注意**：為避免觸發 API 速率限制，腳本在每次備份後會自動暫停 5 秒。大量備份時請預留足夠時間。
- **`mkdocs.yml`**：網站生成配置文件 (MkDocs Configuration)
  - 定義網站結構、主題及多語言支援 (i18n) 設定
