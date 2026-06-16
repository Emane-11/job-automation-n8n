# Job Automation with n8n 

An automated job search engine and real-time application assistant built with n8n, Apify, OpenAI, and Google Workspace integrations. This project transforms a manual job search into an optimized, data-driven career funnel.

## 📁 Repository Contents

This repository is organized into ready-to-import n8n workflow canvas files (.json) and their step-by-step setup guides:

### 1. Job Automation
* **Indeed Job Automation.json**: The ready-to-import n8n workflow template containing the complete Indeed scraper ingestion lines, grading configurations, exact Drive deduplication filters, and logging steps.
* **LinkedIn Job Automation.json**: The accompanying workspace layout mapping custom data variables extracted from LinkedIn's platform framework straight to the automated drafting engine.
* **Indeed & LinkedIn Job Automation Setup Guide.md**: Full blueprint for automated web scraping, job matching, automated Google Drive deduplication, and customized cover letter drafting.

### 2. Email Job Classifier
* **Email Job Classifier.json**: The live production configuration mapping real-time unread Gmail triggers to AI classification engine and high-importance Discord webhook filters.
* **Email Job Classifier Setup Guide.md**: Blueprint for an automated Gmail classification system that parses recruiter responses and pushes real-time status alerts to Discord.


## 🛠️ System Architecture

### 1. The Job Acquisition & Processing Pipeline
This daily tracking workflow extracts open roles from job boards, scores them using LLMs against your professional background, and automatically prepares your application assets.

```
                              +-----------------------+
                              |  Daily / Manual Test  |
                              +-----------+-----------+
                                          |
                              +-----------v-----------+
                              |  Launch Indeed Scraper|
                              +-----------+-----------+
                                          |
                              +-----------v-----------+
                              | Extract Scraped Jobs  |
                              +-----------+-----------+
                                          |
                              +-----------v-----------+
                              |      Limit (10)       |
                              +-----------+-----------+
                                          |
                              +-----------v-----------+
                              | Score Job Description |
                              +-----------+-----------+
                                          |
                                /---------v---------\
                               /  Filter by Score?   \
                              <                       >
                               \     (Score >= 5)    /
                                \---------+---------/
                                          |
                         +----------------+----------------+
                         | True                            | False
                +--------v-----------+           +---------v-----------+
                | Prepare Target File|           |   Log Application1  |
                +--------+-----------+           +---------------------+
                         |
                +--------v-----------+
                | Check Drive Dups   |
                +--------+-----------+
                         |
                /--------v---------\
               /   File Needs      \
              <     Creation?       >
               \   (No Dup ID)     /
                \--------+--------/
                         |
        +----------------+----------------+
        | True                            | False
+-------v-----------+            +--------v-----------+
|    Get Resume     |            | Skip - Processed   |
+-------+-----------+            +--------------------+
        |
+-------v-----------+
|Draft Cover Letter |
+-------+-----------+
        |
+-------v-----------+
| Copy Template     |
+-------+-----------+
        |
+-------v-----------+
| Insert AI Content |
+-------+-----------+
        |
+-------v-----------+
| Log Appl + Letter |
+-------------------+
```

### 2. The Recruiter Response & Triage Pipeline

A real-time Gmail monitoring network that classifies incoming career emails into structured categories and ensures urgent responses are never delayed.
* **Automated Gmail Categorization**: Labels messages into *Applications*, *Interviews*, *Offers*, *Rejections*, or *Recommendations*.
* **Discord Push Engine**: Connects directly via webhook to alert you instantly about high-priority items (Interviews, Offers, and Rejections).

```
                          +-----------------------+
                          |  Gmail Trigger Node   |
                          |  (Unread / Inbox)     |
                          +-----------+-----------+
                                      |
                          +-----------v-----------+
                          | AI Email Classifier   |
                          | (Structured Parser)   |
                          +-----------+-----------+
                                      |
                          +-----------v-----------+
                          | Route by Category     |
                          | (Switch: 0 to 5)      |
                          +-----------+-----------+
                                      |
   +-----------+-----------+----------+----------+-----------+
   | 0         | 1         | 2        | 3        | 4         | 5
+------v-----+ +---v--------+ +v---------+ +v--------+ +v--------+ +v-------+
|  Apply     | | Interviews | |  Offers  | |Reject'ns| |Recommen' | | Ignore  |
|  Label     | |  Label     | |  Label   | | Label   | |  Label   | | (No-op) |
+------------+ +-----+------+ +----+----+  +---+-----+ +----------+ +---------+
                     |             |           |
                     +-------------+-----------+
                                   |
                                   | (High Priority Branch)
                                   v
                         +-------------------+
                         |   Discord Alert   |
                         | (Webhook Channel) |
                         +-------------------+

```

## 🏗️ Getting Started

### Prerequisites
* An active account on **n8n** 
* An **Apify** account with an integrated token balance ($5 free monthly)
* An **OpenAI** or **OpenRouter** API key
* Pre-configured **Google Workspace Documents** (job listings database sheet, cover letter template, and resume doc)

### Brief Installation
1.  **Configure Folders**: Build five identical labels (`Applications`, `Interviews`, `Offers`, `Rejections`, `Recommendations`) inside your target Gmail dashboard.
2.  **Import Canvas Workspace**: Copy and paste your workflow JSON layouts into your n8n design workspace.
3.  **Link Account Connections**: Re-authenticate your native nodes using your saved Google OAuth2, Apify, and API credentials.
4.  **Calibrate Search Parameters**: Modify the keyword expressions and geographic bounds in your scraper profiles to target your relevant market sector.
5.  **Activate Triggers**: Click on **Publish** to publish the workflow so it runs automatically.
