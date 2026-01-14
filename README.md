# Threat Intelligence RSS Automation – Daily Feed to Teams
**This project is under maintenance**

## General Description

This project implements an automated **Threat Intelligence pipeline** built in **n8n**, with the goal of:

- Collecting daily cybersecurity news from multiple RSS feeds
- Storing them in a centralized repository
- Automatically analyzing them using AI
- Publishing only relevant and approved articles to Microsoft Teams

The system is divided into two independent but complementary workflows, which must be executed in the following order:

1. RSS Daily Feed  
<img width="1381" height="356" alt="image" src="https://github.com/user-attachments/assets/7a72cbad-a3a4-4c9f-a67a-c8d43f6c0187" />

2. Feed to Teams/Telegram/slack (whatever you want)
<img width="1735" height="347" alt="image" src="https://github.com/user-attachments/assets/c556b883-b17d-4e09-9f55-b3077e233a4f" />


---

## System Architecture

RSS Feeds → RSS Daily Feed → Data Table  
                              ↓  
                        Feed to Teams  
                              ↓  
                      AI Threat Analysis  
                              ↓  
                   Teams/Slack/Telegram/etc

---

## 1. RSS Daily Feed

### Objective

Collect cybersecurity news published during the day, filter them by date, and store them in a central table for later analysis.

### How it works

1. Scheduled execution  
   - Runs automatically every day at 11:00 via a Cron node.

2. RSS feed ingestion  
   - Consumes multiple sources, for example:
     - BleepingComputer
     - The Hacker News

3. Results unification  
   - Articles from all feeds are merged into a single stream.

4. Date normalization  
   - The `isoDate` field is converted to standard ISO format.

5. Time-based filtering  
   - Only articles are processed that:
     - Have a valid date
     - Were published after the start of the current day

6. Persistence  
   - Each article is inserted into a Data Table with the following fields:
     - Title
     - Link
     - Summary
     - Approve = false
     - processed = false

Result: a daily queue of articles pending analysis.

---

## 2. Feed to Intel

### Objective

Analyze the collected articles, assess their threat level and relevance, and automatically publish the most relevant ones to Microsoft Teams/Slack/Telegram/etc.

### How it works

1. Article retrieval  
   - Rows are fetched from the Data Table.

2. Initial filtering  
   - Only articles that meet the following conditions are processed:
     - Approve = true
     - processed = false

3. Content retrieval  
   - The article link is accessed via HTTP.
   - The main content is extracted using HTML parsing (article tag).

4. AI analysis  
   - An AI Agent acts as a Threat Intelligence Analyst and evaluates:
     - Article summary
     - Attack prerequisites
     - Threat actors
     - Victimology (industries, countries, or regions)
     - Capability, Opportunity, and Intent
     - IOCs and huntable artifacts
     - Attack techniques explained in non-technical language

   The threat is classified as:
   - Potential
   - Impending
   - Insubstantial

5. Status update  
   - The AI analysis result is stored in the table.
   - The article is marked as:
     - processed = true
     - Approve = false

6. Publishing  
   - The analyzed content can be automatically sent to Microsoft Teams as a structured message.

---

## Final Outcome

The system delivers:

- A curated daily Threat Intelligence feed
- Articles analyzed and classified by risk
- Enriched information with technical and strategic context
- Content ready for:
  - SOC teams
  - Threat Hunters
  - CTI Analysts
  - Executive consumption

---

## Data Table – Schema

| Field     | Description |
|----------|-------------|
| Title    | Article title |
| Link     | Original URL |
| Summary  | RSS feed excerpt |
| Sent       | AI-generated analysis |
| Approval  | Manual approval control |
| DailyArticle | Prevents reprocessing |

---

## Requirements

- Functional n8n instance
- Access to:
  - Public RSS feeds
  - OpenAI API
- Basic knowledge of:
  - Automation with n8n

---

## Possible extensions

- AI-based automatic approval
- Integration with SIEM or SOAR platforms
- Risk scoring
- CTI metrics dashboard
- Evolution into a full Threat Intelligence RAG platform
