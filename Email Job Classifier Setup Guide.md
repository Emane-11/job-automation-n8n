# Email Job Classifier Setup Guide

This workflow automatically categorizes incoming emails using an OpenRouter LLM, applies native Gmail labels, and sends instant alerts to Discord for high-priority career updates.

## Workflow Blueprint

<br>

![alt text](<Screenshot 2026-06-15 at 22.00.10.png>)

<br>

The workflow layout follows this simple routing logic:

- **Gmail Trigger**: Fires instantly on a new unread email.

- **AI Email Classifier**: Extracts text, evaluates intent using an OpenRouter model, and outputs a clean category number via the Structured Output Parser.

- **Route by Category**: A switch node that splits the pipeline into 6 distinct paths (0 to 5).

- **Gmail Labels**: Automatically tags the email based on the category path.

- **Discord Alert**: Sends an instant notification if the email is an Interview, Offer, or Rejection.


## Quick Prerequisites

Before activating the canvas, ensure you have these elements ready:

- **n8n Connections**: Connected Gmail OAuth2 account and a Discord Channel Webhook URL.

- **OpenRouter Account**: An active API key with a few cents of credit.

- **Gmail Labels**: Manually create these 5 exact labels in your Gmail account before running the script: `Applications`, `Interviews`, `Offers`, `Rejections`, and `Recommendations`.


## Node-by-Node Setup

### 1. Gmail Trigger

- **Trigger On**: Message Received

- **Filters**: Set to watch your `INBOX` and check for `unread` messages only.

### 2. AI Email Classifier Core

This block consists of three connected nodes working as a single engine:

- **Main Node**: Set your system prompt to analyze the email body and assign a category ID.

- **OpenRouter Chat Model**: Select a fast model like `gpt-4o-mini`, or budget-friendly model like `meta-llama/llama-3-8b-instruct` or `google/gemini-flash`.

- **Structured Output Parser**: Enforce a strict JSON schema so the AI only returns a single category.

### 3. Route by Category (Switch)
Set the mode to **Rules**. Create 6 output routing rules checking the `category` variable from the AI node:

| Branch | Rule Condition | Target Node Action | Routes to Discord? |
| :--- | :--- | :--- | :--- |
| **0** | Value equals `0` | Apply label: `Applications` | No |
| **1** | Value equals `1` | Apply label: `Interviews` | **Yes** |
| **2** | Value equals `2` | Apply label: `Offers` | **Yes** |
| **3** | Value equals `3` | Apply label: `Rejections` | **Yes** |
| **4** | Value equals `4` | Apply label: `Recommendations` | No |
| **5** | Value equals `5` | `Ignore: Other` (No-op / End) | No |

### 4. Discord Alert
Connect the output ports of Gmail Label nodes **1, 2, and 3** directly to this single Discord node. 

Paste your channel Webhook URL and use this scannable layout for the message payload:

```text
🚨 New Job Update!  
📬 Status: {{ $('AI Email Classifier').item.json.output.category }} 
📧 Subject: {{ $('Gmail Trigger').item.json.subject }}  
Check your inbox for details.
```

### System Scope, Limitations, & Best Practices

#### 1. Pre-Creating Folders and Label Architecture

The automated labeling nodes cannot dynamically generate a folder path inside Gmail if it does not exist when the API call fires. Before activating your automation pipeline, open your native Gmail dashboard interface. Create five custom labels named exactly after your target tracks: Applications, Interviews, Offers, Rejections, and Recommendations. Once built, refresh your n8n workspace so the node parameters can pull them from the API dropdown list.

#### 2. Managing API Costs and Token Overhead

While processing emails via OpenRouter is highly cost-effective, long corporate email threads can consume significant input tokens. To prevent bloated processing costs, use the parameters inside your Gmail Trigger node to truncate the body content. Passing only the first 2,000 to 4,000 characters of clean message text provides more than enough semantic data for an LLM to accurately figure out the category while keeping token usage minimal.

#### 3. Handling AI Classification Drift

No machine learning classification system is 100% flawless. Occasionally, a highly nuanced recruiter email might sit on the borderline between an application confirmation and an interview setup. To prevent critical updates from slipping under the radar due to a rare classification mistake, build a routine check into your schedule. Dedicate a few minutes every few days to check your Recommendations or Applications folders manually, ensuring that no misclassified messages require urgent responses.

#### 4. Resolving Discord Trigger Node Disconnections

If you notice that emails are successfully receiving labels inside your Gmail account but your Discord channel remains completely empty, check and ensure that all three independent branch lines (Interviews, Offers, and Rejections) are linked to the input port of the Discord Alert node. If any line is disconnected, that specific category will process silently within Gmail without ever triggering an external alert. Also, check that your Discord Webhook configuration permissions are active and have not been reset by server administrators.
