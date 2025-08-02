# 🧩 Generate Yearly Report for Vendor (Work Item 4)  
*UiPath RPA Developer Advanced – Assignment 2*  

Automates the Yearly Report process for vendor transactions in **ACME System 1**, using the official guideline for Work Item 4. It collects monthly reports for each vendor, consolidates them into a single Excel file, and uploads it back—leveraging **REFramework** and UiPath Orchestrator for scalable execution.

---

## 📌 Business Case  
Each vendor requires a yearly summary Excel report generated from monthly invoice data. The process involves:  
- Scraping Work Items of type **WI4**  
- Downloading monthly reports (previous calendar year)  
- Combining them into one file named `Yearly-Report-[Year]-[TaxID].xlsx`  
- Uploading the completed report  
- Marking the Work Item as **Completed** with a comment `Uploaded with ID [UploadID]`  
- Repeating until all WI4 items are processed  
All these steps are defined in the official Process Design Document¹. :contentReference[oaicite:1]{index=1}

---

## 🤖 Workflow Overview  
1. **Dispatcher**  
   - Scrape all UI Work Items (to enqueue page numbers / WI IDs as transactions)  
   - Use the **REFramework** Dispatcher state machine to queue each Work Item  
2. **Performer**  
   - For each transaction:  
     1. Open details page → extract `Tax ID`  
     2. Navigate to **Download Monthly Report** panel → download all files for *(previous calendar year)*  
     3. Consolidate them into a single Excel file `Yearly-Report-<Year>-<TaxID>.xlsx`  
     4. Upload via **Upload Yearly Report** section → capture unique **Upload ID**  
     5. Return to Work Item → set status to **Completed**, comment: `Uploaded with ID <UploadID>`  
3. Repeat for each WI4 transaction until the queue is empty, or all items are completed.

---

## 🗂 Project Structure 
```Generate-Yearly-Report-For-Vendor/
│
├── ACME System1/
│   ├── Common/                          # Shared workflows (Launch, Login, Logout, etc.)
│   │   ├── ACMESystem1_CloseWindow.xaml
│   │   ├── ACMESystem1_EmailInvalidCreds.xaml
│   │   ├── ACMESystem1_Launch.xaml
│   │   ├── ACMESystem1_Login.xaml
│   │   └── ACMESystem1_Logout.xaml
│   │
│   ├── Dispatcher/                      # Dispatcher bot – adds WI4 items to Orchestrator queue
│   │   ├── ACMESystem1_OpenWorkItemsPage.xaml
│   │   └── ACMESystem1_ReadWiStable.xaml
│   │
│   ├── Performer/                       # Performer bot – processes each Work Item
│   │   ├── ACMESystem1_CloseWindow.xaml
│   │   ├── ACMESystem1_CreateFolderDownloadedReport.xaml
│   │   ├── ACMESystem1_DownloadReport.xaml
│   │   ├── ACMESystem1_ExtractTaxID.xaml
│   │   ├── ACMESystem1_FillDownloadReportsFields.xaml
│   │   ├── ACMESystem1_FillUploadReportFields.xaml
│   │   ├── ACMESystem1_NavigateToDownloadMonthlyReports.xaml
│   │   ├── ACMESystem1_NavigateToEachW1.xaml
│   │   ├── ACMESystem1_NavigateToUploadYearlyReport.xaml
│   │   └── ACMESystem1_UpdateWorkItem.xaml
│
├── Data/
│   ├── Input/                           # Downloaded data for each TaxID (monthly reports)
│   ├── Output/                          # Final `Yearly-Report-*.xlsx` exports
│   ├── Documentation/                  # PDD, reference documents, screenshots
│   ├── Excel/                           # Contains `ACMESystem1_PrepareToUpload.xaml` and Config files
│   └── Exceptions_Screenshots/         # Screenshots captured on exception
│
├── Framework/                          # REFramework base workflows
│   ├── CloseAllApplications.xaml
│   ├── Dispatcher.xaml
│   ├── GetTransactionData.xaml
│   ├── InitAllApplications.xaml
│   ├── InitAllSettings.xaml
│   ├── KillAllProcesses.xaml
│   ├── Process.xaml
│   ├── RetryCurrentTransaction.xaml
│   ├── SetTransactionStatus.xaml
│   └── TakeScreenshot.xaml
│
├── Screenshots/                        # GIFs or UI snapshots of bot in action
├── project.json                        # UiPath project manifest file
├── README.md                           # Project documentation (you're reading it!)
└── .gitignore                          # Files/folders excluded from version control
```
---

## ⚙️ Framework Setup & Dependencies

- Based on **UiPath REFramework** template (Dispatcher + Performer patterns) :contentReference[oaicite:3]{index=3}
- Install via **Manage Packages**:
  - `UiPath.System.Activities`
  - `UiPath.Excel.Activities`
  - `UiPath.UIAutomation.Activities`
  - **Optional**: `UiPath.Credentials.Activities`, `UiPath.Mail.Activities` (if implementing email notifications)

> Ensure your **Config.xlsx** in `Data/Excel/` contains:  
> - `QueueName` (e.g., `InHouse_Process4`)  
> - `ACMESystem1_URL`  
> - `CredentialName`  
> - `Year` (previous year)  
> - `MaxRetryNumber` (e.g., 2)  
> These must match the PDD and Orchestrator setup :contentReference[oaicite:4]{index=4}

---

## 🚀 How to Run This Project

1. **Configure**:
   - Save credentials in **Orchestrator** as Asset
   - Create a **Queue** matching `QueueName`
   - Place `Config.xlsx` in the `Data/Config.xlsx/` folder

2. **Run Dispatcher**:
   - Opens ACME System 1
   - Gathers page numbers or work item identifiers
   - Adds them to the Orchestrator queue

3. **Run Performer**:
   - Processes each queued item in turn
   - Downloads, consolidates, uploads, updates status and comments

4. **Verify**:
   - Work Items appear as **Completed** in ACME System 1
   - Excel file `Yearly-Report-[Year]-[TaxID].xlsx` generated for each vendor  
   - No failed transactions in Orchestrator queue

---

## 🧠 Error Handling & Logging

- Built-in retry logic per REFramework – handles login errors, page navigation timeouts, download failures
- If **required monthly file is missing**, skip and continue (per PDD; up to ~2–3 months missing allowed) :contentReference[oaicite:5]{index=5}
- For unknown/unhandled exceptions:
  - Take screenshot (`TakeScreenshot.xaml`)
  - Log exception message
  - (Optional) **Send email** from `exceptions@[yourdomain].com`
- **Configurable** retry count (`MaxRetryNumber`) to avoid infinite loops

---

👤 Author / Maintainer
Karim Adel, RPA Developer – UiPath Certified

Developed alongside UiPath RPA Developer Advanced Assignment 2

Contact: kariimadell97@gmail.com 