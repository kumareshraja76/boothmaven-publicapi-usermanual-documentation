---
title: "Salesforce Integration"
sidebarTitle: "Salesforce Integration"
description: "Learn how to connect BoothMaven with Salesforce and synchronize contacts and other supported data."
icon: "cloud"
---

# BoothMaven & Salesforce Integration — User Guide & Setup Manual

> **Effortless Event Lead Synchronization**: Connect BoothMaven with Salesforce once via secure OAuth, and your captured contacts, accounts, trade show campaigns, booth meetings, and voice/text notes will synchronize automatically in real time — **zero manual CSV exports or imports required**.

---

## Table of Contents

1. [Overview & Key Benefits](#1-overview--key-benefits)
2. [How the Integration Works](#2-how-the-integration-works)
3. [One-Time Setup: Connecting BoothMaven to Salesforce (OAuth 2.0)](#3-one-time-setup-connecting-boothmaven-to-salesforce-oauth-20)
4. [What Gets Synchronized Automatically?](#4-what-gets-synchronized-automatically)
   - [4.1 Automatic Contact & Account Matching](#41-automatic-contact--account-matching)
   - [4.2 Campaign & Campaign Member Attribution](#42-campaign--campaign-member-attribution)
   - [4.3 Two-Way Meeting & Calendar Sync](#43-two-way-meeting--calendar-sync)
   - [4.4 Voice Memos & Text Notes Sync](#44-voice-memos--text-notes-sync)
   - [4.5 Content Engagement Signals](#45-content-engagement-signals)
5. [Deduplication & Data Integrity](#5-deduplication--data-integrity)
6. [Managing Custom Field Mappings & Qualification Questions](#6-managing-custom-field-mappings--qualification-questions)
7. [Required Salesforce Permissions](#7-required-salesforce-permissions)
8. [Troubleshooting & Frequently Asked Questions](#8-troubleshooting--frequently-asked-questions)

---

## 1. Overview & Key Benefits

Trade show leads often sit in spreadsheets for days or weeks before reaching your sales team. BoothMaven's direct Salesforce integration eliminates this lag entirely.

```
┌─────────────────────────────────┐                 ┌─────────────────────────────────┐
│        BOOTHMAVEN APP           │                 │         SALESFORCE CRM          │
│                                 │                 │                                 │
│  • Badge Scans & Lead Forms     │ ──────────────► │  • Contacts & Accounts          │
│  • Trade Show Event             │ ──────────────► │  • Campaigns & Campaign Members │
│  • Booth Meetings               │ ◄─────────────► │  • Events / Tasks (2-Way)       │
│  • Text Notes & Voice Memos     │ ──────────────► │  • Notes & Audio Files          │
│  • Brochure & QR Views          │ ──────────────► │  • Activity Tasks & Signals     │
└─────────────────────────────────┘                 └─────────────────────────────────┘
                   Real-Time Synchronization (Zero CSV Uploads)
```

### Key Highlights:

- **Immediate Speed-to-Lead**: Attendees scanned at the booth appear in Salesforce within seconds.
- **Instant Event ROI Attribution**: Automatically creates or links a Salesforce Campaign and marks leads as Campaign Members (`Responded`).
- **Complete Conversation History**: Text notes and audio voice memos recorded on the floor attach directly to the Contact record.
- **Two-Way Meeting Sync**: Meetings scheduled in BoothMaven appear in Salesforce calendars; meeting outcomes updated by sales reps in Salesforce sync back to BoothMaven.
- **Automatic Session Healing**: Connection refreshes automatically in the background without needing re-authentication.

---

## 2. How the Integration Works

Once OAuth authentication is authorized, BoothMaven continuously monitors for activity in your event booth. Every time a booth representative performs an action, BoothMaven dispatches an intelligent, asynchronous background sync job to your Salesforce org:

1. **Email Lookup**: BoothMaven checks if the attendee's email already exists in Salesforce.
2. **Contact & Account Action**: If found, it enriches the existing record; if new, it creates a new Contact and matches/creates the Account.
3. **Campaign Attribution**: Links the Contact to the Salesforce Campaign representing the trade show.
4. **Activity & Notes Attachment**: Schedules meetings, uploads notes, and logs digital engagement signals directly under the Contact's activity timeline.

---

## 3. One-Time Setup: Connecting BoothMaven to Salesforce (OAuth 2.0)

Setting up the integration takes less than 2 minutes and requires no developer assistance.

### Step-by-Step Instructions:

1. **Log in to BoothMaven**:
   - Open the [BoothMaven Web Dashboard](https://app.boothmaven.com/) as an Organization Administrator.
2. **Navigate to Integrations**:
   - From the left sidebar, click **Settings**, then select **Integrations**.
   - Locate the **Salesforce** card and click **Configure** (or **Connect**).
3. **Select Your Environment**:
   - **Production / Developer Edition**: Select this if you are connecting to your live Salesforce organization (`login.salesforce.com`).
   - **Sandbox**: Select this if you want to test in a Salesforce staging/sandbox environment (`test.salesforce.com`).
4. **Authorize with Salesforce**:
   - Click **Connect with Salesforce**.
   - You will be redirected to the official Salesforce login page. Enter your Salesforce admin or integration user credentials.
   - On the OAuth consent screen, click **Allow** to grant BoothMaven access to synchronize data.
5. **Confirmation**:
   - You will be returned to the BoothMaven dashboard with a green badge: **"Connected & Active"**.
   - The screen will display your connected Salesforce User and Organization ID.

> **Note:** BoothMaven uses secure OAuth 2.0 with refresh tokens. You will not need to re-enter passwords when Salesforce sessions expire; BoothMaven renews tokens automatically in the background.

---

## 4. What Gets Synchronized Automatically?

---

### 4.1 Automatic Contact & Account Matching

When a rep scans a badge or submits a contact card:

- **Contact Creation / Update**:
  - Scans search for an existing Contact by **Email**.
  - If found, existing fields are preserved and event context (Booth, Rep, Event Name, Lead Score) is enriched.
  - If new, a Contact is created with `LeadSource = "boothmaven"`.
- **Account Matching**:
  - BoothMaven extracts the company name and corporate email domain.
  - Searches Salesforce for a matching `Account.Name` or domain website.
  - Automatically associates the Contact to the appropriate company Account.

#### Standard Fields Synchronized:

| BoothMaven Field    | Salesforce Target | Salesforce Field API Name                           |
| :------------------ | :---------------- | :-------------------------------------------------- |
| First Name          | Contact           | `FirstName`                                         |
| Last Name           | Contact           | `LastName`                                          |
| Email Address       | Contact           | `Email`                                             |
| Phone Number        | Contact           | `Phone` / `MobilePhone`                             |
| Title / Designation | Contact           | `Title`                                             |
| Company             | Contact / Account | `BM_Company__c` / `Account.Name`                    |
| Lead Source         | Contact           | `LeadSource` (`boothmaven`)                         |
| Lead Score          | Contact           | `BM_Lead_Score__c`                                  |
| Captured By Rep     | Contact           | `BM_Captured_By__c`                                 |
| Capture Method      | Contact           | `BM_Capture_Method__c` (e.g. `badge_scan`, `kiosk`) |

---

### 4.2 Campaign & Campaign Member Attribution

Every event configured in BoothMaven connects directly to the Salesforce Campaign hierarchy:

- **Automatic Campaign Association**: BoothMaven links to an existing Salesforce Campaign or auto-creates a Campaign named after your event (e.g. _"CES 2026 — Booth #5012"_).
- **Instant Campaign Member Creation**: Every attendee scanned at the booth is added as a `CampaignMember` with status **`Responded`**.
- **ROI Visibility**: Marketing teams can immediately view total event reach, pipeline influenced, and won revenue directly in Salesforce Campaign reports.

---

### 4.3 Two-Way Meeting & Calendar Sync

When booth staff book an on-site demo or follow-up call with an attendee:

- **Calendar Event Created**: BoothMaven creates a Salesforce `Event` or `Task` record linked directly to the attendee's Contact record (`WhoId`).
- **Complete Meeting Details**: Includes meeting title, date, start/end time, duration, meeting pod/room location, and agenda notes.
- **Two-Way Status Sync**: If an account executive updates the meeting status in Salesforce (e.g., _Completed_, _Rescheduled_, or _No-Show_), the outcome flows back into BoothMaven so booth managers have accurate daily reporting.

---

### 4.4 Voice Memos & Text Notes Sync

Booth conversations contain invaluable qualitative context that standard forms miss:

- **Text Notes**: Rep notes taken in BoothMaven immediately sync as standard Salesforce `Note` records attached to the Contact.
- **Audio Voice Memos**: Reps can record 30-second audio voice memos right after talking to an attendee. BoothMaven uploads the audio file as a Salesforce `ContentVersion` document linked to the Contact, allowing field reps and account executives to listen to the prospect's exact requirements and tone.

---

### 4.5 Content Engagement Signals

When booth visitors interact with your digital assets:

- When a prospect scans your digital card, downloads a PDF product brochure, or watches an embedded demo video after the show, BoothMaven logs a high-priority `Task` in Salesforce.
- Shows the exact document viewed, duration spent reading, and timestamp so your sales reps can follow up while interest is peak.

---

## 5. Deduplication & Data Integrity

To keep your Salesforce database clean:

1. **Email-First Deduplication**: BoothMaven checks `Email = '{attendee_email}'` before creating any new Contact record.
2. **Safe Field Updates**: BoothMaven does not overwrite existing telephone numbers, titles, or company names unless specifically configured. It safely updates event-specific fields (e.g., latest event visited, lead score, qualification responses).
3. **Audit Trail**: Every synchronized record logs a sync transaction in BoothMaven (`CrmLog`) with the Salesforce Record ID and timestamp for total traceability.

---

## 6. Managing Custom Field Mappings & Qualification Questions

Does your booth team ask qualifying survey questions (e.g., _"What is your purchasing timeline?"_, _"What is your current vendor?"_)?

- In **BoothMaven Dashboard &rarr; Settings &rarr; Integrations &rarr; Salesforce &rarr; Field Mapping**, you can map each qualification question directly to any custom field on your Salesforce Contact or Campaign Member object (e.g., `Purchasing_Timeline__c`, `Budget_Approved__c`).

---

## 7. Required Salesforce Permissions

For the connection to synchronize smoothly, the connecting Salesforce user profile must have the following standard permissions:

- **API Enabled**: Permission to use the Salesforce REST API.
- **Standard Object Permissions**:
  - **Contacts**: Read, Create, Edit.
  - **Accounts**: Read, Create, Edit.
  - **Campaigns & Campaign Members**: Read, Create, Edit.
  - **Events & Tasks (Activities)**: Read, Create, Edit.
  - **Notes & Files (ContentVersion)**: Read, Create.

---

## 8. Troubleshooting & Frequently Asked Questions

### Q: How do I know if a lead synchronized successfully?

**A:** In the BoothMaven web portal, go to **Contacts**. Next to each contact, a Salesforce cloud icon indicates sync status:

- 🟢 **Green**: Synchronized successfully (hover to view the Salesforce Contact ID).
- 🟡 **Yellow**: In queue for delivery.
- 🔴 **Red**: Failed (click to see the exact error message from Salesforce, e.g., missing required field or validation rule).

### Q: What happens if a Salesforce validation rule blocks a contact?

**A:** If a custom Salesforce validation rule rejects a record (for example, a mandatory custom field), BoothMaven records the error in the activity log and retains the lead. Once the field mapping or validation rule is adjusted, click **Retry Sync** in the BoothMaven dashboard to push the lead without data loss.

### Q: Can I disconnect or switch to another Salesforce instance?

**A:** Yes. In **Settings &rarr; Integrations &rarr; Salesforce**, click **Disconnect**. You can then reconnect to a different Sandbox or Production environment at any time.

### Q: Does BoothMaven count against my Salesforce API limits?

**A:** BoothMaven optimizes API usage with efficient payload batching and targeted query calls, using negligible API call volume even for high-traffic booths with thousands of scans per day.

---

## Need Support?

- **BoothMaven Integration Support**: [support@boothmaven.com](mailto:support@boothmaven.com)
- **Knowledge Base & Help**: [https://boothmaven.com/help](https://boothmaven.com/help)
