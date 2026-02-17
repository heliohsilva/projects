# Lazy Mail

## Overview

Lazy Mail is an automated agent that filters and summarizes important emails from Microsoft Outlook using LLMs and delivers a daily briefing via Telegram.

The system addresses inbox overload by reducing noise from newsletters, spam, and phishing attempts, ensuring that relevant messages (such as professional opportunities) are not missed.

---

## Problem

Over the years, my primary email account accumulated:

- Old newsletters
- Recurring promotional emails
- Phishing attempts
- Inconsistent spam filtering (including false positives)

The default spam filter is not always reliable — I have missed important communications due to emails being incorrectly classified.

Daily manual triage became time-consuming and inefficient.

---

## Solution

I developed an automated workflow that:

1. Runs every morning
2. Calls the Microsoft Graph API to collect emails from the previous day
3. Sends the retrieved data to an LLM to perform:
   - Relevance classification
   - Promotional content filtering
   - Summary generation
4. Delivers the output as a newsletter-style message via a Telegram bot

The result is a concise morning briefing containing only the relevant emails from the previous day.

---

## Architecture

- Daily trigger (cron)
- Request to Microsoft Outlook via Graph API
- Processing and filtering through an LLM
- Automated delivery via Telegram bot

Simplified flow:

# WORKFLOW

---

## Technologies Used

- n8n (workflow orchestration)
- Microsoft Graph API
- LLM (classification and summarization)
- Telegram Bot API
- Self-hosted environment (Home Lab)

---

## Technical Decisions

- Used n8n to accelerate prototyping and simplify visual workflow maintenance.
- Leveraged LLM-based processing instead of rule-based filters to allow dynamic adaptation to different email patterns.
- Chose Telegram as the delivery interface for its speed, accessibility, and seamless integration into daily routines.

---

## Technical Challenges

- Designing filters that minimize false negatives without discarding relevant emails.
- Controlling LLM operational costs based on email volume.
- Fine-tuning prompts to ensure concise and objective outputs.

---

## Results

- Eliminated daily manual email triage.
- Significantly reduced the risk of missing important messages.
- Low operational cost (a few cents per day).
- Stable system running daily in a self-hosted environment.

---

## Key Learnings

- Practical integration with the Microsoft Graph API.
- Prompt engineering for classification and summarization tasks.
- Designing automation systems focused on real productivity gains.
