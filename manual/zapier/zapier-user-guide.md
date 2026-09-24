---
title: "Zapier Integration"
sidebarTitle: "Zapier Integration"
description: "Learn how to connect BoothMaven with Zapier and use the integration API to automate workflows."
icon: "bolt"
---

# BoothMaven & Zapier Integration — User Guide

Welcome to the **BoothMaven + Zapier Integration Guide**. This guide explains how to connect your BoothMaven account to Zapier and automate your event workflow across 6,000+ business applications (including Google Sheets, HubSpot, Salesforce, Slack, Mailchimp, Google Calendar, and more) — **no coding or technical expertise required**.

---

## Table of Contents

1. [What Can You Do with BoothMaven & Zapier?](#1-what-can-you-do-with-boothmaven--zapier)
2. [How to Connect BoothMaven to Zapier (One-Time Setup)](#2-how-to-connect-boothmaven-to-zapier-one-time-setup)
3. [BoothMaven Triggers (Send Data from BoothMaven to Your Apps)](#3-boothmaven-triggers-send-data-from-boothmaven-to-your-apps)
   - [Trigger 1: New Contact](#trigger-1-new-contact)
   - [Trigger 2: New Meeting](#trigger-2-new-meeting)
4. [BoothMaven Actions (Send Data from Your Apps into BoothMaven)](#4-boothmaven-actions-send-data-from-your-apps-into-boothmaven)
   - [Action 1: Create Contact](#action-1-create-contact)
   - [Action 2: Create Meeting](#action-2-create-meeting)
5. [Step-by-Step Walkthrough: Building Your First Zap](#5-step-by-step-walkthrough-building-your-first-zap)
   - [Example: Send New Booth Contacts to a Google Sheet](#example-send-new-booth-contacts-to-a-google-sheet)
6. [Popular Pre-Built Workflow Ideas](#6-popular-pre-built-workflow-ideas)
7. [Frequently Asked Questions & Troubleshooting](#7-frequently-asked-questions--troubleshooting)

---

## 1. What Can You Do with BoothMaven & Zapier?

When you attend a trade show or conference with BoothMaven, your booth staff capture badge scans, digital business cards, and schedule prospect meetings.

With Zapier, that data immediately syncs into the tools your company already uses:

- **Instant Lead Follow-Up**: Stream scanned leads in real time into your CRM (HubSpot, ActiveCampaign, Pipedrive) or an email sequence.
- **Team Notifications**: Alert booth managers or territory reps in Slack or Microsoft Teams whenever a VIP lead is scanned.
- **Backup to Spreadsheets**: Keep a live, auto-updating Google Sheet of every attendee met at the show.
- **Calendar Sync**: Automatically sync booth meetings directly to Google Calendar or Outlook.
- **Inbound Lead Pre-Registration**: Push RSVP lists from Typeform, Eventbrite, or your landing page directly into BoothMaven as pre-loaded contacts.

---

## 2. How to Connect BoothMaven to Zapier (One-Time Setup)

Connecting your BoothMaven account takes under 60 seconds using secure **OAuth 2.0 single-sign-on**.

### Steps to Connect:

1. Log in to your [Zapier account](https://zapier.com/).
2. In the left navigation, click **Apps** (or **Connected Accounts**), then click **+ Add connection**.
3. Search for **BoothMaven** and select it.
4. A secure login window will pop up:
   - Enter your **BoothMaven Email** and **Password**.
   - Click **Sign In**.
5. You will see an authorization screen: _"BoothMaven is requesting permission to access your contacts and meetings."_ Click **Authorize**.
6. The window will close automatically. In your Zapier dashboard, your connection will appear with your account email address (e.g. `jane@example.com`).

> **Tip:** You only need to connect once! All Zaps you build will automatically reuse this secure connection.

---

## 3. BoothMaven Triggers (Send Data from BoothMaven to Your Apps)

Triggers start a Zap when an event happens in BoothMaven.

---

### Trigger 1: New Contact

- **What it does**: Runs automatically every time a new lead is captured in BoothMaven (via badge scan, QR code scan, digital business card exchange, or manual entry).
- **Speed**: Instant (Real-Time Webhook).
- **Data Fields Sent to Your Apps**:
  - **First Name** (`first_name`)
  - **Last Name** (`last_name`)
  - **Email Address** (`email`)
  - **Phone Number** (`phone`)
  - **Company Name** (`company`)
  - **Job Title / Designation** (`job_title`)
  - **Event ID & Event Name** (`event_id`, `event_name`)
  - **Date & Time Captured** (`created_at`)

#### Example Use Cases:

- Add attendee to a HubSpot list tagged with the event name.
- Add a row to a centralized Google Sheet for the sales team.
- Send an automated _"Great to meet you at [Event Name]!"_ personalized email via Gmail or Mailchimp.

---

### Trigger 2: New Meeting

- **What it does**: Runs automatically whenever a meeting is booked in BoothMaven (scheduled by a booth representative or booked by an attendee via your digital card/kiosk link).
- **Speed**: Instant (Real-Time Webhook).
- **Data Fields Sent to Your Apps**:
  - **Meeting Title / Topic** (`meeting_name`)
  - **Meeting Date & Time** (`meeting_date`, `meeting_time`)
  - **Duration** (`meeting_duration` in minutes)
  - **Meeting Format** (`meeting_format` — virtual, inPerson, or hybrid)
  - **Meeting Location / Video Link** (`meeting_location`)
  - **Meeting Agenda / Notes** (`meeting_agenda`)
  - **Public Meeting Link** (`meeting_url`)
  - **Primary Attendee Details**: Name, Email, Phone, Company
  - **Event Name & Event ID** (`event_name`, `event_id`)

#### Example Use Cases:

- Create a detailed Google Calendar or Outlook event with attendee details in the description.
- Send an SMS confirmation to the attendee via Twilio.
- Create a Salesforce or HubSpot Task assigned to the booth sales rep.

---

## 4. BoothMaven Actions (Send Data from Your Apps into BoothMaven)

Actions allow other apps to create or schedule records inside BoothMaven.

---

### Action 1: Create Contact

- **What it does**: Creates a new contact in your BoothMaven contact book.
- **When to use**: When you collect RSVPs or pre-show inquiries in a webform (Typeform, Webflow, Jotform) and want your booth staff to have those contacts readily available in the BoothMaven mobile app.
- **Fields You Can Map**:
  | Field Name | Required? | Description |
  | :--- | :--- | :--- |
  | **First Name** | **Yes** | Contact's first name. |
  | **Last Name** | No | Contact's last name or surname. |
  | **Email** | No | Primary email address. |
  | **Phone Number** | No | Mobile or office phone number. |
  | **Company** | No | Company or organization name. |
  | **Job Title** | No | Job title or designation. |

---

### Action 2: Create Meeting

- **What it does**: Schedules a new meeting on your BoothMaven event schedule.
- **When to use**: When a client books a demo slot through Calendly, HubSpot Meetings, or your website before or during the trade show, and you want that meeting to show up in BoothMaven.
- **Fields You Can Map**:
  | Field Name | Required? | Description |
  | :--- | :--- | :--- |
  | **Meeting Name** | **Yes** | Subject or title (e.g. _"Product Demo - Acme Corp"_). |
  | **Meeting Date** | **Yes** | Date in `YYYY-MM-DD` format (e.g. `2026-10-15`). |
  | **Meeting Time** | **Yes** | Time (e.g. `14:00` or `2:00 PM`). |
  | **Attendee Email** | No | Email of the contact to link with this meeting. |
  | **Duration (Minutes)** | No | Length in minutes (default is 30). |
  | **Meeting Format** | No | Choose `virtual`, `inPerson`, or `hybrid`. |
  | **Meeting Location** | No | Physical booth number or video link (Google Meet / Zoom). |
  | **Meeting Agenda** | No | Topics, notes, or specific demo requirements. |
  | **Timezone** | No | Timezone of the event (e.g. `America/New_York`, `UTC`). |
  | **Priority** | No | `low`, `medium`, or `high`. |

---

## 5. Step-by-Step Walkthrough: Building Your First Zap

Let's build a practical Zap: **Save Every New Booth Contact to a Google Sheet in Real Time**.

```
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│       STEP 1: TRIGGER           │       │        STEP 2: ACTION           │
│           BoothMaven            │ ────► │          Google Sheets          │
│          "New Contact"          │       │    "Create Spreadsheet Row"     │
└─────────────────────────────────┘       └─────────────────────────────────┘
```

### Step 1: Set Up the Trigger in Zapier

1. In your Zapier dashboard, click **+ Create Zap**.
2. Click the **Trigger** block and search for **BoothMaven**.
3. Under **Event**, choose **New Contact**, then click **Continue**.
4. Select your connected **BoothMaven account** and click **Continue**.
5. Click **Test trigger**. Zapier will fetch a recent contact from your BoothMaven account to verify the connection. Click **Continue with selected record**.

### Step 2: Set Up the Action in Google Sheets

1. In the next block, search for **Google Sheets**.
2. Under **Event**, choose **Create Spreadsheet Row** and click **Continue**.
3. Select your Google account and pick your spreadsheet (e.g., _"Event Leads 2026"_).
4. Map the spreadsheet columns to your BoothMaven fields:
   - **Full Name** &rarr; Select `First Name` + `Last Name`
   - **Email** &rarr; Select `Email`
   - **Company** &rarr; Select `Company`
   - **Title** &rarr; Select `Job Title`
   - **Event** &rarr; Select `Event Name`
   - **Captured Date** &rarr; Select `Created At`
5. Click **Test step**. Check your Google Sheet to confirm the new row appeared!
6. Click **Publish** to turn on your Zap.

🎉 **You're all done!** Every time a booth rep scans a badge or adds a contact in BoothMaven, it instantly appears in your Google Sheet.

---

## 6. Popular Pre-Built Workflow Ideas

| Goal                      | Trigger App  | Trigger Event   | Action App          | Action Event                      |
| :------------------------ | :----------- | :-------------- | :------------------ | :-------------------------------- |
| **Instant Lead Scoring**  | BoothMaven   | New Contact     | **HubSpot**         | Create / Update Contact           |
| **Real-Time Team Alert**  | BoothMaven   | New Contact     | **Slack**           | Send Channel Message              |
| **Calendar Reservation**  | BoothMaven   | New Meeting     | **Google Calendar** | Quick Add / Create Event          |
| **Pre-Show Registration** | **Typeform** | New Submission  | BoothMaven          | Create Contact                    |
| **Calendly Demo Sync**    | **Calendly** | Invitee Created | BoothMaven          | Create Meeting                    |
| **Email Welcome Blast**   | BoothMaven   | New Contact     | **Mailchimp**       | Add Subscriber to Tagged Audience |

---

## 7. Frequently Asked Questions & Troubleshooting

### Q: How quickly does data transfer between BoothMaven and my other apps?

**A:** Instantly! BoothMaven uses real-time Webhooks (REST Hooks). As soon as a lead is saved or a meeting is scheduled, BoothMaven pushes the data to Zapier within seconds.

### Q: What happens if our booth loses internet connection at the venue?

**A:** If booth devices are offline, the BoothMaven app safely saves the lead locally. As soon as the device reconnects to Wi-Fi or cellular data, leads sync to BoothMaven servers and all your Zaps fire automatically.

### Q: Can multiple team members use the same Zapier connection?

**A:** Yes. Once your account is connected, all contacts and meetings captured by your team under your BoothMaven workspace are processed by your Zaps.

### Q: Why does my Zap say "Connection Expired" or "Unauthorized"?

**A:** OAuth security tokens may periodically need re-authorization. If prompted, go to **Zapier -> Apps -> BoothMaven**, click **Reconnect**, and log in again. Your active Zaps will resume immediately.

---

## Need Assistance?

- **BoothMaven Support**: [support@boothmaven.com](mailto:support@boothmaven.com)
- **Website**: [https://boothmaven.com](https://boothmaven.com)
- **Zapier Help Center**: [https://help.zapier.com](https://help.zapier.com)
