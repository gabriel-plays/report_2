# SAPS Crime Statistics — Direct Download URLs

**Source:** South African Police Service (SAPS)
**Main portal:** https://www.saps.gov.za/services/crimestats.php
**Downloads directory:** https://www.saps.gov.za/services/downloads/

---

## ✅ Confirmed .xlsx Files (indexed by search engines)

| Year | File | Direct URL |
|------|------|-----------|
| 2021/2022 | Annual Crime Stats | https://www.saps.gov.za/services/downloads/Crime-Statistics-2021_2022-web.xlsx |
| 2020/2021 | Annual Crime Stats | https://www.saps.gov.za/services/downloads/Crime-Statistics-2020_2021-Release.xlsx |
| 2023/2024 | 3rd Quarter | https://www.saps.gov.za/services/downloads/2023-2024_-_3nd_Quarter_WEB.xlsx |
| 2021/2022 | 2nd Quarter | https://www.saps.gov.za/services/downloads/second_quarter_2021_2022_release.xlsx |
| 2023/2024 | Annual Financial Year | https://www.saps.gov.za/services/downloads/2024/2023-2024%20_Annual_Financial%20year_WEB.xlsx |

---

## 🔍 Likely xlsx Files (based on SAPS naming conventions — try these)

### 2022/2023
- https://www.saps.gov.za/services/downloads/2022-2023_-_2nd_Quarter_WEB.xlsx
- https://www.saps.gov.za/services/downloads/2022-2023_-_3rd_Quarter_WEB.xlsx
- https://www.saps.gov.za/services/downloads/2022-2023_-_4th_Quarter_WEB.xlsx
- https://www.saps.gov.za/services/downloads/Crime-Statistics-2022_2023-web.xlsx

### 2021/2022 (additional quarters)
- https://www.saps.gov.za/services/downloads/third_quarter_2021_2022_release.xlsx
- https://www.saps.gov.za/services/downloads/fourth_quarter_2021_2022_release.xlsx
- https://www.saps.gov.za/services/downloads/first_quarter_2021_2022_release.xlsx

### 2023/2024 (additional quarters)
- https://www.saps.gov.za/services/downloads/2023-2024_-_1st_Quarter_WEB.xlsx
- https://www.saps.gov.za/services/downloads/2023-2024_-_2nd_Quarter_WEB.xlsx
- https://www.saps.gov.za/services/downloads/2023-2024_-_4th_Quarter_WEB.xlsx

### 2024/2025
- https://www.saps.gov.za/services/downloads/2024/2024-2025_-_1st_Quarter_WEB.xlsx
- https://www.saps.gov.za/services/downloads/2024/2024-2025_-_2nd_Quarter_WEB.xlsx
- https://www.saps.gov.za/services/downloads/2024/2024-2025_-_3rd_Quarter_WEB.xlsx

### 2019/2020
- https://www.saps.gov.za/services/downloads/Crime-Statistics-2019_2020-web.xlsx

---

## 📋 How to Download All Files

**Option 1 — Manual download:**
Visit https://www.saps.gov.za/services/crimestats.php and click the Excel download links for each quarter.

**Option 2 — Direct links:**
Copy and paste any URL from the table above directly into your browser address bar to download.

**Option 3 — Bulk download (Windows PowerShell):**
```powershell
$urls = @(
  "https://www.saps.gov.za/services/downloads/Crime-Statistics-2021_2022-web.xlsx",
  "https://www.saps.gov.za/services/downloads/Crime-Statistics-2020_2021-Release.xlsx",
  "https://www.saps.gov.za/services/downloads/2023-2024_-_3nd_Quarter_WEB.xlsx",
  "https://www.saps.gov.za/services/downloads/second_quarter_2021_2022_release.xlsx",
  "https://www.saps.gov.za/services/downloads/2024/2023-2024%20_Annual_Financial%20year_WEB.xlsx"
)
foreach ($url in $urls) {
  $filename = Split-Path $url -Leaf
  Invoke-WebRequest -Uri $url -OutFile ".\$filename"
  Write-Host "Downloaded: $filename"
}
```

---

## 📂 Alternative Sources

If SAPS links are unavailable, try:

- **openAFRICA:** https://open.africa/en/dataset/?tags=crime
  (Community-maintained SAPS data extracts)

- **PoliceData ZA:** https://www.policedata.online/
  (Interactive dashboard, may offer CSV exports)

- **CrimeHub:** https://crimehub.org/topics/crime-statistics
  (Aggregated SAPS data with download options)

---

*Compiled: March 2026. URLs based on confirmed search engine indexing and SAPS file naming conventions.*
