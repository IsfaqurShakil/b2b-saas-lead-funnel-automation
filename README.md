# B2B SaaS Lead Funnel Automation (Prototype)

A lead capture, scoring, and follow-up automation for a B2B SaaS demo-request funnel. Built with free tools only: n8n, Google Sheets, Telegram, Gmail, Google Tag Manager, and GA4.

> **Demo project.** The product ("ComplyDesk") is fictional. No real company is represented. All lead data in this project is demo data.

**Live landing page:** https://isfaqurshakil.github.io/b2b-saas-lead-funnel-automation/

## The problem

In a B2B SaaS funnel, demo requests from very different buyers often get the same treatment. High-intent leads wait too long, lead data is copied by hand (causing duplicates and errors), and when an automation step fails, nobody finds out. This project is a prototype of a funnel that fixes those gaps. These gaps are assumptions to be confirmed in discovery with a real team.

## What it does

1. A visitor opens the landing page (optionally through a UTM-tagged link) and fills in the demo form.
2. The form sends the data to an n8n webhook. GTM fires a GA4 `generate_lead` event.
3. n8n checks the Google Sheet for the same email. Duplicates are stopped.
4. n8n calculates a lead score and a label (hot, warm, or cold).
5. The lead is saved to the Google Sheet.
6. The next step depends on the label:
   - **Hot:** a Telegram message asks a human to approve or reject. Only after approval does the follow-up email go out through Gmail, and the Sheet row is updated. A rejection updates the row and sends nothing.
   - **Warm:** an onboarding email is sent automatically.
   - **Cold:** a nurture email is sent automatically.
7. A separate error workflow sends a Telegram alert when any step fails.

```
Landing page (UTM) -> GTM / GA4 event
        |
        v
   n8n webhook -> duplicate check -> lead score -> save to Google Sheet
                       |                                  |
                  (duplicate: stop)          hot / warm / cold
                                                  |
        hot: Telegram approval -> Gmail -> Sheet update (or reject -> Sheet update)
        warm: onboarding email
        cold: nurture email

   Any failure -> error workflow -> Telegram alert
```

## Lead scoring

| Signal | Points |
|---|---|
| Company size 200+ / 51-200 / 11-50 / 1-10 | 25 / 20 / 15 / 5 |
| Role: safety, EHS, or compliance / operations or founder / other | 25 / 15 / 5 |
| Need: set up or replace library, or multi-site / just exploring | 20 / 5 |
| Timeline: within 1 month / 1-3 months / just researching | 20 / 10 / 0 |
| Work email (not a free provider such as Gmail) | 10 |

Total 70-100 is **hot**, 40-69 is **warm**, 0-39 is **cold**. The weights are assumptions. A live version would tune them using real conversion data.

## Tools (all free tier)

| Purpose | Tool |
|---|---|
| Landing page hosting | GitHub Pages |
| Automation | n8n (self-hosted, reached through an ngrok tunnel during testing) |
| Lead storage | Google Sheets |
| Alerts and approval | Telegram bot |
| Emails | Gmail (OAuth2) |
| Tracking | Google Tag Manager and GA4 |

## What is tracked

- GA4 records page views and a `generate_lead` event when the form is submitted.
- Hidden form fields save `utm_source`, `utm_medium`, `utm_campaign`, and the page URL with each lead.
- No ads were run. Conversion tracking is set up and ads-ready, and there are no Google Ads or LinkedIn Ads results.

## Testing

Test cases and the bugs found and fixed are recorded in the Test Log tab of the Google Sheet. Examples of bugs fixed: Gmail API not enabled, a service account without a Gmail mailbox, template variables shown as raw text, a wrong node order in the duplicate check, and a Telegram button that used a localhost link.

The reject path is built but is not yet covered in the test log.

## Failure points and human approval

- Duplicates are detected before a lead is saved.
- Any workflow error triggers a Telegram alert through a separate error workflow.
- Hot lead emails are sent only after a human approves them.

## Known limitations of this prototype

- The workflow runs on a personal computer through a tunnel, so the form works only while n8n and the tunnel are running.
- The webhook URL is visible in the page source. A live version would add request validation.
- Lead data sits in a Google Sheet. A live version would use a CRM, with consent text and a privacy policy.
- Score weights are assumptions and have not been checked against real conversions.

## Demo vs live

| Area | This demo | A live version |
|---|---|---|
| Product and data | Fictional, demo data | Real product and real leads, with consent |
| Hosting | Local n8n and a tunnel | Always-on server with a fixed domain |
| Landing page | Standalone page | The company's own site or form sends data to the same webhook |
| Storage | Google Sheet | CRM |
| Scoring | Fixed rules in code | Rules in a config table, tuned with real data |
| Emails | Personal Gmail | Company mailbox or email service, with unsubscribe link |

## Repository contents

- `index.html`: the landing page with the form, hidden UTM fields, and the GTM snippet
- `README.md`: this file
- n8n workflow exports (JSON) will be added to this repository

## Author

Isfaqur Rahman Shakil
