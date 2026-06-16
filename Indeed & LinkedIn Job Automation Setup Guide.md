# Indeed & LinkedIn Job Automation Setup Guide

Automate your job search, application tracking, and AI-generated cover letters using an integrated pipeline powered by n8n, Apify, OpenAI, Google Docs, Google Drive, and Google Sheets.

## Workflow Overview

Searching and applying for jobs manually can quickly become a repetitive, time-consuming numbers game. This workflow transforms that linear grind into an automated processing loop by executing an end-to-end data pipeline. The system autonomously scrapes fresh job listings from Indeed or LinkedIn, evaluates every description against your professional resume, scores each opportunity based on your specific career criteria, and filters out low-match positions. For high-scoring roles, it runs a programmatic deduplication check before drafting hyper-personalized cover letters, compiling polished text files in Google Drive, and recording every processed opportunity in a centralized Google Sheets database.

The ultimate objective of this architecture is not to completely automate your job applications, but rather to establish a highly capable intelligent assistant. It filters out low-yield vacancies, handles heavy copy-drafting lifting, and keeps your entire career outreach systematically organized. By deploying this infrastructure, you will have an automated pipeline that runs daily, leaving you with ready-to-review cover letters for the most relevant positions in your market.

## Workflow Architecture

The automated architecture executes in a sequential data processing line, branching out into smart routing tracks based on cognitive scoring thresholds.


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


### Visual  Blueprint

![alt text](<Screenshot 2026-06-16 at 22.27.40.png>)

## Essential Prerequisites

This pipeline is designed to be entirely approachable for individuals without deep programming backgrounds. However, you must configure a series of external cloud services and document frameworks before initializing the setup.

### 1. Core Automation & AI Accounts

- **n8n Automation Account**: Sign up for an account on n8n. The platform provides a 14-day free trial, which is an ideal window for deploying, configuring, and validating this workflow. This trial includes integrated free OpenAI credits, allowing you to test complex prompts, calibrate job scoring metrics, and run sample cover letter generation cycles without spending money or linking an external API key. Note that platform subscription structures can shift over time, so verify current limits directly on the official website.

- **Apify Web Scraping Account**: Sign up for a free account via the official Apify registration portal.  Apify gives you **$5 of free platform credits every single month**. For personal application pipelines, this balance is more than sufficient to scrape hundreds of live job postings, making it an incredibly cost-effective collection asset.

- **OpenAI API Access**: For production use beyond the trial credits, you will need a dedicated OpenAI API key. The pipeline balances processing speed, context comprehension, and infrastructure billing by leveraging highly efficient large language models such as GPT-4o or GPT-4o-mini.


### 2. Documents & Database Infrastructure

Before importing the workflow json canvas, download the provided templates and place them inside your Google Workspace environment:

- **Job Applications Database**: 
	Link: https://docs.google.com/spreadsheets/d/1k4nSv9rp3nX7MgIx-RbXm5-7Nkmt6Ja-lweEVktSSh0/edit?usp=sharing
	
	Copy the `Job Listings Database` file and save it into your Google Sheets workspace. Ensure it contains target tracking headers like Link, Title, Company, Location, Score, and Status.

- **Cover Letter Template**: 
	Link: https://docs.google.com/document/d/165e0fKFhgE_YoemRk7gHi33wzliCTMR8FHBhRfT_Bic/edit?usp=sharing
	
	Copy the `Cover Letter Template` file into your Google Docs workspace, ensuring your personal styling preference is clean and ready for script injections.

- **Master Candidate Resume**: Upload an up-to-date, text-dense copy of your current CV directly into Google Docs so that the language models can extract your professional history.

## Detailed Step-by-Step Configuration

### Step 1: Configure the Workflow Trigger

The pipeline supports two entry points depending on your operational phase. For daily automated production deployment, use the **Daily Check Trigger** node. This runs as a system cron job, typically scheduled to fire every day at 8:00 AM to parse fresh early-morning job postings. While building, testing, or adjusting prompt logic, utilize the **Manual Test Trigger** node to instantly fire the entire network with a single dashboard click.

![alt text](<Screenshot 2026-06-16 at 11.29.18.png>)

### Step 2: Configure the Job Scraper

This node interfaces directly with cloud actors to crawl job boards for listings matching your precise career trajectories. You can deploy the **Indeed Jobs Scraper** actor or the **LinkedIn Jobs Scraper** actor or both depending on your target marketplace. Note that certain metadata schemas native to Indeed may vary slightly if switching exclusively to LinkedIn tracking data.

These are the Used scrapers for this project:

Indeed jobs scrapper :  `Indeed jobs scraper [PPR]` by borderline
	link: https://apify.com/borderline/indeed-scraper

LinkedIn jobs scrapper : `Linkedin Jobs Scraper` by curious_coder
	link: https://apify.com/curious_coder/linkedin-jobs-scraper

To install this component, open your Apify workspace, locate your desired scraper actor page, configure your inputs, click save then copy the generated target input JSON. Then, open n8n, add an Apify Actor node, connect your API credentials, and paste that JSON block into the configuration field.

![alt text](<Screenshot 2026-06-16 at 11.24.01.png>)

![alt text](<Screenshot 2026-06-16 at 11.24.35.png>)

![alt text](<Screenshot 2026-06-16 at 11.25.30.png>)


#### Critical Search Parameters

You must customize the input JSON block to reflect your target market:

- **Keywords**: Input structured query phrases such as `"Data Analyst" OR "Business Analyst" OR "Junior Data Scientist" OR "BI Analyst"`.
    
- **Geographic Focus**: Set explicit location terms (e.g., `Casablanca`, `Remote`, `Morocco`, `Europe`, or `United Kingdom`).
    
- **Work Mode & Experience**: Filter for parameters like `Remote Only`, `Hybrid`, or `On-site`, alongside experience tier tags like `Entry Level` or `Junior`.

### Step 3: Extract and Ingest Scraped Jobs

The **Extract Scraped Jobs** node connects to the temporary data dataset generated by your cloud scraper. It functions as an ingestion translator, transforming unstructured array listings into individual, cleanly indexed JSON rows within n8n. Without this step, downstream nodes would receive an invalid, compound data wrapper that stops parallel loop operations. This parsing mechanism systematically isolates the core attributes of each listing, creating separate variables for the job title, company name, raw description text, application URL, location, and listed salary boundaries.

### Step 4: Establish Data Processing Limits

The **Limit** node serves as a financial defense checkpoint in your pipeline. Because every single job description routed to a Large Language Model consumes API tokens, passing unstructured data pools without constraints can cause runaway billing costs. During the early sandbox and testing phase, set this value strictly between `3` and `10` items per run. Once your prompts are perfectly calibrated and you are confident in your filtering accuracy, you can increase this ceiling to `50+` rows for daily production runs.

### Step 5: Evaluation via AI Job Scoring

The **Generate a Score** node acts as the primary cognitive gatekeeper for your career funnel. It reads the full, unstructured text of the job post, ingests your raw master resume document, and applies structured scoring rules to evaluate compatibility.

#### Core Evaluation Metrics

> - **Resume Match**: Checks if the target technical requirements align closely with your existing toolsets and technical proficiencies.
>    
> - **Experience Match**: Cross-references qualification demands against your background to verify capability.
>    
> - **Preference Match**: Validates whether the role adheres to your strict constraints regarding remote work flexibility, skills match, target salary lines, etc...
>     

The output from this node is forced into a strict JSON structure containing a math rating out of 10 along with an explicit textual reasoning string:

JSON

```
{
  "score": 8,
  "reasoning": "Strong match with core data analytics skillsets and aligns perfectly with regional location preferences."
}
```

The effectiveness of this entire pipeline rests on how well you tailor this scoring prompt to mirror your real-world career requirements.

### Step 6: Filter Opportunities by Score

The **Filter by Score** conditional node reads the math metric computed by the previous AI stage. If an opportunity scores highly (e.g., `Score >= 5`), it passes through the **True** branch directly into the cover letter writing pipeline. If a listing fails to meet your minimum threshold (e.g., `Score < 5`), it is routed into the **False** branch. This secondary path triggers a Google Sheets logging action that saves the rejected job details to a history file. This records skipped vacancies so the system never wastes API credits re-evaluating the same low-yield job posts on future scraper cycles.

### Step 7: Implement Duplicate Detection and Conditional Creation

To prevent the pipeline from duplicating effort or cluttering your workspace with redundant documents, a native two-step validation chain handles duplicate mitigation right after a job passes the qualification score:

1. **Prepare Target Filename (Code Node):** This lightweight processing step evaluates incoming job details and dynamically formats the strict text pattern of your expected file: `Cover letter_[Job_Title]_[Company_Name]`.

<br>

![alt text](<Pasted image 20260616223415.png>)

2. **Check Drive for Duplicates (Google Drive Node):** Rather than using unpredictable custom scripts, this step utilizes an advanced direct search query via the native Google Drive integration. It safely executes an explicit target search directly against your live file library.

<br>

![alt text](<Pasted image 20260616223242.png>)

### Step 8: Retrieve Candidate Resume

The **Get Resume** node securely communicates with Google Docs to download your master CV file. Ensure you toggle the **"Simplify"** parameter to **Enabled** inside this node's settings. Doing so strips away complex HTML styling wrappers or layout tables, exposing clean, text-dense info to the downstream copywriting engines to optimize your token budget.

![alt text](<Screenshot 2026-06-16 at 11.42.01.png>)


### Step 9: Generate Hyper-Personalized Cover Letters

The **Draft Cover Letter Content** node uses an AI Agent to craft a tailored pitch. This node merges your structural framework, your resume highlights, and the specific pain points hidden within the job description into an impact-driven narrative.

![alt text](<Screenshot 2026-06-16 at 12.05.22.png>)

#### Formatting Instructions for the Agent

- **Avoid Generic Openers**: Ban generic text blocks like _"My name is..."_ or _"I am writing to apply for..."_. Start immediately with a unique observation or strategic industry insight regarding the company's market space.
    
- **Quantifiable Achievements**: Emphasize concrete historical achievements using a precise action-driven formula: `Action + Context + Measurable Result`.
    
- **Tone & Volume**: Maintain a highly professional, motivated, and objective tone, ensuring the letter remains white-space friendly and under a maximum boundary of 250 words.
    

To ensure consistent data routing downstream, a **Structured Output Parser** node must be attached directly to the canvas slot of this AI Agent. This parser enforces a rigid JSON schema, packaging the written letter into manageable fields like `company_name`, `job_title`, and `cover_letter` so subsequent file automation nodes can interpret the text without errors.


### Step 10: Compile Documents and Log Applications

Once the text fields are parsed, the **Copy Cover Letter Template** node creates a fresh copy of your master styled Google Doc inside your designated application folder. Be careful to input your exact master template file ID string here, as a mismatch can halt document creation.

![alt text](<Screenshot 2026-06-16 at 22.23.45.png>)

Next, the **Insert AI Content into Document** node uses a "Find and Replace Text" action to locate placeholders inside your new file (such as `{company_name}` or `{cover_letter}`) and replaces them with the parsed text fields generated by your AI Agent.

![alt text](<Screenshot 2026-06-16 at 11.54.27.png>)

Finally, the **Log Application + Cover Letter** node records the successful run inside your centralized Google Sheets workbook. Set the **Operation** parameter at the top of this node to **Append or Update Row**, using the unique job link as your matching key. This allows n8n to cleanly insert a brand-new line item for fresh opportunities, or seamlessly refresh an existing row if a job profile is re-processed, mapping the application date, job title, company name, AI qualification score, and your generated document link.


![alt text](<Pasted image 20260616222857.png>)

<br>

## Operational Scope & Limitations

### 1. Token Budgets & OpenAI Billing

Every interaction with an external language model consumes input and output tokens. Large batch files containing multi-page descriptions and extensive CV text blocks can deplete credit balances quickly. To maintain strong cost controls, utilize optimized models like `gpt-4o-mini`, keep your master resume text concise, and rely heavily on the early AI scoring node to screen out low-match jobs before generating cover letters.

### 2. Guarding Against AI Hallucinations

Generative AI models can occasionally misinterpret text data, invent metrics, or exaggerate your experience with a specific toolset. Because of this, you must **never fully automate applications without human review**. Treat this pipeline as a specialized administrative assistant: always open the generated document from your tracking sheet to audit the content, refine the wording, and verify its accuracy before submitting.

### 3. Web Scraping & API Maintenance

Major job boards like Indeed change their underlying HTML code frameworks frequently. When these system adjustments occur, web scraping elements can break or return incomplete data profiles. Check your background execution logs regularly to verify that your Apify actors are updated and extracting fields correctly.

### 4. Platform Execution Ceilings

If you run this architecture on a standard cloud free tier, be aware of active workflow ceilings and processing timeout constraints. Since the web scraper node uses a **"Wait for Finish"** option that pauses operations while crawling the web, long cycles can trigger execution timeouts. If your workflow stalls, reduce the scraper's maximum row processing limit or migrate your pipeline to a self-hosted n8n community instance for unrestricted runtime flexibility.

## Final Summary

This automation framework transforms an exhausting, manual job hunt into a centralized, semi-automated processing system. By maintaining clear handoffs between web scrapers, cognitive scoring nodes, duplicate checkers, and file generators, the pipeline continuously uncovers new listings, vets them against your career goals, and prepares customized materials for you.

Once you calibrate your search terms, fine-tune your scoring parameters, and refine your AI writing prompts, this system operates as a powerful personal assistant. It eliminates hours of repetitive manual data entry, keeps your pipeline organized, and allows you to focus your energy entirely on preparing for interviews at companies that truly match your background.