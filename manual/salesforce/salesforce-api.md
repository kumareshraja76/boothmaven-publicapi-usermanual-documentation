---
title: "Salesforce Integration API"
sidebarTitle: "Salesforce Integration"
description: "Learn how to connect BoothMaven with Salesforce and synchronize contacts and other supported data."
icon: "cloud"
---

# BoothMaven - Salesforce Integration API Documentation

> **Reference URL**: [https://www.boothmaven.com/integration/salesforce/](https://www.boothmaven.com/integration/salesforce/)  
> **API Version**: Salesforce REST API v60.0 (Spring '24 / Summer '24)  
> **Supported Plans**: **Essential ($149/mo)** & **Business ($399/mo)** _(Not available on Capture)_  
> **Contact / Support**: [hello@boothmaven.com](mailto:hello@boothmaven.com) &bull; [BoothMaven.com](https://boothmaven.com)

---

## Overview

BoothMaven connects event lead capture, badge scanning, and attendee interactions directly with Salesforce CRM in real time without batch imports or CSV files.

Unlike conventional lead capture tools that push flat contact rows, BoothMaven integrates natively with the **Salesforce Campaign Object**:

- **One Campaign per Event**: Auto-generated or linked when an event is configured in BoothMaven.
- **One Campaign Member per Captured Lead**: Holds event context, booth number, rep, capture method, lead score, and qualifying answers.
- **Contact & Account Creation/Matching**: Contacts matched by email; Accounts matched by company domain/name with Apollo.io company intelligence.
- **Two-Way Meeting Sync (Business Plan)**: Meetings sync as Salesforce Tasks/Events; rep updates in Salesforce flow back to BoothMaven.
- **Content Engagement Signals**: Spec sheet, brochure, and pricing views sync as Tasks to trigger Salesforce Flows.

---

## Table of Contents

1. [Authentication & Connection (OAuth 2.0)](#1-authentication--connection-oauth-20)
2. [Data Model & Object Hierarchy](#2-data-model--object-hierarchy)
3. [API Endpoints & Integration Requests](#3-api-endpoints--integration-requests)
   - [3.1 Contact & Account Sync](#31-contact--account-sync)
   - [3.2 Campaign & Campaign Member Sync](#32-campaign--campaign-member-sync)
   - [3.3 Meeting & Activity Sync (Two-Way)](#33-meeting--activity-sync-two-way)
   - [3.4 Voice & Text Notes Sync](#34-voice--text-notes-sync)
   - [3.5 Content Engagement Signals](#35-content-engagement-signals)
4. [Field Mapping Specifications](#4-field-mapping-specifications)
5. [Deduplication & Conflict Handling](#5-deduplication--conflict-handling)
6. [Asynchronous Trigger Points & Execution Architecture](#6-asynchronous-trigger-points--execution-architecture)
   - [Sync vs. Async Trigger Matrix](#sync-vs-async-trigger-matrix)
   - [Job Dispatch Pipeline & Observers](#job-dispatch-pipeline--observers)
   - [Audit Trail & State Management (CrmLog)](#audit-trail--state-management-crmlog)
7. [Salesforce Flow Automation Triggers](#7-salesforce-flow-automation-triggers)

---

## 1. Authentication & Connection (OAuth 2.0)

BoothMaven connects to Salesforce using **OAuth 2.0 Web Server Flow** (Authorization Code Grant).

### Base URLs

- **Production Login**: `https://login.salesforce.com`
- **Sandbox Login**: `https://test.salesforce.com`
- **REST API Path**: `https://{instance_url}/services/data/v60.0/`

### OAuth Endpoints

| Step               | Method | URL                                     | Description                                     |
| :----------------- | :----- | :-------------------------------------- | :---------------------------------------------- |
| **Authorize**      | `GET`  | `{login_url}/services/oauth2/authorize` | Initiates user consent and OAuth code grant     |
| **Token Exchange** | `POST` | `{login_url}/services/oauth2/token`     | Exchanges authorization code for tokens         |
| **Token Refresh**  | `POST` | `{login_url}/services/oauth2/token`     | Refreshes expired session using `refresh_token` |

#### Required Scopes

- `api`: Access and manage Salesforce data via REST API.
- `refresh_token` / `offline_access`: Allows background synchronization when user is not active.
- `full` / `id`: User identity and profile validation.

#### Token Refresh Payload (`POST /services/oauth2/token`)

```http
POST /services/oauth2/token HTTP/1.1
Host: login.salesforce.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&client_id=YOUR_CONSUMER_KEY
&client_secret=YOUR_CONSUMER_SECRET
&refresh_token=YOUR_REFRESH_TOKEN
```

#### Automatic Session Healing

If any Salesforce API call returns an HTTP `401 Unauthorized` with `errorCode: INVALID_SESSION_ID`, BoothMaven automatically refreshes the token and transparently replays the request.

---

## 2. Data Model & Object Hierarchy

```
               ┌───────────────────────────────┐
               │      Salesforce Campaign      │
               │   (One per BoothMaven Event)  │
               └───────────────┬───────────────┘
                               │
                               ▼
┌──────────────────┐   ┌───────────────────────────────┐   ┌──────────────────┐
│Salesforce Account│◄──┤      Salesforce Contact       │──►│ Campaign Member  │
│(Company Domain)  │   │     (One per Event Lead)      │   │(Event Attribution│
└──────────────────┘   └───────────────┬───────────────┘   │ & Qualifying)    │
                                       │                   └──────────────────┘
            ┌──────────────────────────┼──────────────────────────┐
            ▼                          ▼                          ▼
 ┌────────────────────┐     ┌────────────────────┐     ┌────────────────────┐
 │  Salesforce Event  │     │  Salesforce Note / │     │  Salesforce Task   │
 │   / Meeting Task   │     │   ContentVersion   │     │  (Content Signal)  │
 │  (2-Way Sync)      │     │(Voice & Text Notes)│     │(Doc & Video Views) │
 └────────────────────┘     └────────────────────┘     └────────────────────┘
```

---

## 3. API Endpoints & Integration Requests

### 3.1 Contact & Account Sync

Creates or updates a Salesforce `Contact`. If company information is captured, it matches or creates an `Account`.

- **Endpoint**: `PATCH /services/data/v60.0/sobjects/Contact/{id}` or `POST /services/data/v60.0/sobjects/Contact`
- **Query Check**: `GET /services/data/v60.0/query?q=SELECT+Id+FROM+Contact+WHERE+Email='{escapedEmail}'+LIMIT+1`

#### Request Payload (`Contact`)

```json
{
  "FirstName": "James",
  "LastName": "Porter",
  "Email": "james.porter@medtechsolutions.com",
  "Phone": "+1-415-555-0199",
  "MobilePhone": "+1-415-555-0188",
  "Title": "Chief Medical Officer",
  "MailingCity": "San Francisco",
  "MailingCountry": "United States",
  "LeadSource": "boothmaven",
  "Description": "Status: Connected",
  "BM_Capture_Method__c": "badge_scan",
  "BM_Capture_Source__c": "booth_app",
  "BM_Captured_At__c": "2026-09-14T10:30:00Z",
  "BM_Captured_By__c": "Priya Sharma",
  "BM_Company__c": "MedTech Solutions",
  "BM_website__c": "https://medtechsolutions.com",
  "BM_Industry__c": "Healthcare & Biotechnology",
  "BM_Lead_Score__c": 88,
  "BM_Role__c": "Chief Medical Officer",
  "Returning_visitor__c": false,
  "BM_Social_Profiles__c": "LinkedIn: https://linkedin.com/in/jamesporter\nTwitter: https://twitter.com/drporter",
  "BM_Qualification_Data__c": "Budget approved: Yes — Q2 2026\nDecision authority: Final say\nTimeline: 3-6 months\nCurrent Vendor: Siemens"
}
```

---

### 3.2 Campaign & Campaign Member Sync

Every BoothMaven event creates/updates a Salesforce `Campaign` and links the lead as a `CampaignMember`.

- **Campaign Endpoint**: `POST /services/data/v60.0/sobjects/Campaign`
- **Campaign Member Endpoint**: `POST /services/data/v60.0/sobjects/CampaignMember`

#### Campaign Payload

```json
{
  "Name": "HIMSS 2026 — Booth #3142",
  "Type": "Trade Show",
  "Status": "In Progress",
  "StartDate": "2026-09-14",
  "EndDate": "2026-09-17",
  "Description": "HIMSS Global Health Conference & Exhibition 2026",
  "BM_Booth_Number__c": "#3142",
  "BM_Venue__c": "Orange County Convention Center",
  "BM_Event_Location__c": "Orlando, FL",
  "BM_Attendance_Format__c": "in_person"
}
```

#### Campaign Member Payload

```json
{
  "CampaignId": "7015g000000XyZaAAK",
  "ContactId": "0035g000002WbCdAAK",
  "Status": "Responded"
}
```

---

### 3.3 Meeting & Activity Sync (Two-Way)

Syncs booth meetings as Salesforce `Event` or `Task` records. When sales reps update the meeting status in Salesforce (Completed, Rescheduled, No-Show), the outcome syncs back to BoothMaven.

- **Endpoint**: `POST /services/data/v60.0/sobjects/Event`
- **Update Endpoint**: `PATCH /services/data/v60.0/sobjects/Event/{id}`
- **Multi-Contact Link**: `POST /services/data/v60.0/sobjects/EventRelation`

#### Request Payload (`Event`)

```json
{
  "Subject": "Product Demo · HIMSS 2026",
  "StartDateTime": "2026-09-15T14:30:00Z",
  "EndDateTime": "2026-09-15T15:00:00Z",
  "Location": "Booth #3142 Meeting Pod A",
  "Description": "Deep dive on compliance certifications & API security. Contact wants technical documentation.",
  "WhoId": "0035g000002WbCdAAK",
  "OwnerId": "0055g000001AbCdAAK",
  "IsAllDayEvent": false,
  "ShowAs": "Busy"
}
```

#### EventRelation Payload (Multiple Contacts Link)

```json
{
  "EventId": "00U5g000003EfGhAAK",
  "RelationId": "0035g000002WbCdAAK"
}
```

---

### 3.4 Voice & Text Notes Sync

Booth conversations, voice memo transcriptions, and rep text notes are attached directly to the Salesforce Contact:

- **Text Notes**: Synced as standard Salesforce `Note` (`ParentId`, `Title`, `Body`).
- **Audio Voice Notes**: Uploaded as binary `ContentVersion` records linked to `ContentDocumentLink`.

#### Text Note Payload (`POST /services/data/v60.0/sobjects/Note`)

```json
{
  "ParentId": "0035g000002WbCdAAK",
  "Title": "Meeting Note - Product Demo",
  "Body": "Currently with Siemens. Evaluating alternatives. RFP stage. Wants compliance documentation by Friday."
}
```

#### Audio Voice Note Payload (`POST /services/data/v60.0/sobjects/ContentVersion`)

```json
{
  "Title": "Contact Audio Note - 2026-09-15 14:35",
  "PathOnClient": "recording_1002.wav",
  "VersionData": "UklGRiQAAABXQVZFZm10IBAAAAABAAEA...",
  "FirstPublishLocationId": "0035g000002WbCdAAK"
}
```

---

### 3.5 Content Engagement Signals

When attendees view digital cards, download technical spec sheets, or view brochures after the show, BoothMaven creates a `Task` associated with the Contact.

#### Content Signal Task Payload (`POST /services/data/v60.0/sobjects/Task`)

```json
{
  "WhoId": "0035g000002WbCdAAK",
  "Subject": "📄 Content Signal: Compliance Cert Doc Viewed",
  "Status": "Completed",
  "Priority": "High",
  "ActivityDate": "2026-09-18",
  "Description": "Contact viewed 'Compliance Cert Doc'.\nDuration: 6 minutes 38 seconds\nVisit Count: 2nd visit\nChannel: QR Code scan\nTimestamp: 2026-09-18T16:22:10Z"
}
```

---

## 4. Field Mapping Specifications

| BoothMaven Field    | Salesforce Target Object | Salesforce Field API Name        | Field Type    | Plan Tier         |
| :------------------ | :----------------------- | :------------------------------- | :------------ | :---------------- |
| First Name          | Contact                  | `FirstName`                      | String(40)    | Essential+        |
| Last Name           | Contact                  | `LastName`                       | String(80)    | Essential+        |
| Email Address       | Contact                  | `Email`                          | Email         | Essential+        |
| Phone Number        | Contact                  | `Phone`                          | Phone         | Essential+        |
| Mobile Phone        | Contact                  | `MobilePhone`                    | Phone         | Essential+        |
| Designation / Title | Contact                  | `Title`                          | String(128)   | Essential+        |
| Company Name        | Contact / Account        | `BM_Company__c` / `Account.Name` | String(255)   | Essential+        |
| City                | Contact                  | `MailingCity`                    | String(40)    | Essential+        |
| Country             | Contact                  | `MailingCountry`                 | String(80)    | Essential+        |
| Lead Source         | Contact                  | `LeadSource` (`boothmaven`)      | Picklist      | Essential+        |
| Lead Score          | Contact                  | `BM_Lead_Score__c`               | Number(3,0)   | Essential+        |
| Capture Method      | Contact                  | `BM_Capture_Method__c`           | String(50)    | Essential+        |
| Captured By (Rep)   | Contact                  | `BM_Captured_By__c`              | String(100)   | Essential+        |
| Capture Timestamp   | Contact                  | `BM_Captured_At__c`              | DateTime      | Essential+        |
| Social Profiles     | Contact                  | `BM_Social_Profiles__c`          | LongTextArea  | Essential+        |
| Event Title         | Campaign                 | `Name`                           | String(80)    | Essential+        |
| Booth Number        | Campaign                 | `BM_Booth_Number__c`             | String(50)    | Essential+        |
| Event Dates         | Campaign                 | `BM_Event_Dates__c`              | Date          | Essential+        |
| Meeting Name        | Event / Task             | `Subject`                        | String(255)   | Essential+        |
| Meeting Dates       | Event                    | `StartDateTime` / `EndDateTime`  | DateTime      | Essential+        |
| Meeting Outcome     | Event                    | `ShowAs`                         | Picklist      | Business (2-way)  |
| Qualifying Answers  | Contact / Member         | **Any Custom Field Target**      | Custom Mapped | **Business Plan** |

---

## 5. Deduplication & Conflict Handling

BoothMaven implements a multi-stage deduplication strategy to preserve CRM data integrity:

1. **Email Address Matching**: Before creating a Contact, a SOQL query checks `SELECT Id FROM Contact WHERE Email = '{email}' LIMIT 1`.
2. **Native Duplicate Rule Catching**: If Salesforce throws a `DUPLICATES_DETECTED` error, BoothMaven extracts the existing duplicate record ID from the error response and patches the existing Contact instead of failing.
3. **"Met Again" Visitor Logic**: If a lead is captured at a second event, BoothMaven does **not** create a duplicate Contact; it links the existing Contact as a `CampaignMember` to the new Campaign and logs a "Met Again" activity.

---

## 6. Asynchronous Trigger Points & Execution Architecture

To keep the BoothMaven mobile scanner instant (sub-second response even on spotty trade show Wi-Fi), **all Salesforce API network calls are offloaded asynchronously** using Laravel's queue workers.

### Sync vs. Async Trigger Matrix

| User / System Action             | Trigger Point                          | Execution Mode              | Queue Job / Handler                                          |
| :------------------------------- | :------------------------------------- | :-------------------------- | :----------------------------------------------------------- |
| **OAuth Connect**                | User clicks Connect in Settings        | **Synchronous**             | `SalesforceOAuthController@callback`                         |
| **Connection Test**              | User clicks "Test Connection"          | **Synchronous**             | `SalesforceService@testConnection`                           |
| **Token Refresh**                | Session expired (`INVALID_SESSION_ID`) | **Synchronous** (Auto-heal) | `SalesforceService@refreshAccessToken`                       |
| **Lead Captured at Booth**       | Scanner saves contact                  | **⚡ Asynchronous**         | `CustomerContactsObserver` &rarr; `GlobalIntegrationSyncJob` |
| **Event Created / Edited**       | Event organizer saves event            | **⚡ Asynchronous**         | `EventObserver` &rarr; `GlobalIntegrationSyncJob`            |
| **Attendee Event Check-in**      | Attendee linked to event               | **⚡ Asynchronous**         | `ContactEventObserver` &rarr; `GlobalIntegrationSyncJob`     |
| **Meeting Scheduled**            | Rep books booth meeting                | **⚡ Asynchronous**         | `ContactMeetingObserver` &rarr; `GlobalIntegrationSyncJob`   |
| **Meeting Outcome Updated**      | Rep marks Completed/No-Show            | **⚡ Asynchronous**         | `MeetingObserver` &rarr; `GlobalIntegrationSyncJob`          |
| **Voice / Text Note Added**      | Voice memo transcribed                 | **⚡ Asynchronous**         | `ContactNoteObserver` &rarr; `GlobalIntegrationSyncJob`      |
| **Meeting Note Added**           | Minutes added to meeting               | **⚡ Asynchronous**         | `MeetingNoteObserver` &rarr; `GlobalIntegrationSyncJob`      |
| **Lead Score Recalculated**      | Signals update score                   | **⚡ Asynchronous**         | `LeadScoringEngine` &rarr; `GlobalIntegrationSyncJob`        |
| **Apollo.io Company Enrichment** | Lead domain resolved                   | **⚡ Asynchronous**         | `ProcessEnrichmentJob` &rarr; `GlobalIntegrationSyncJob`     |

### Job Dispatch Pipeline & Observers

```
[Mobile App / Web] ──> [Database Record Inserted / Updated]
                                    │
                                    ▼ (Eloquent Observer Fires)
                      ┌────────────────────────────────────────┐
                      │ CustomerContactsObserver               │
                      │ EventObserver / ContactEventObserver   │
                      │ ContactMeetingObserver / NoteObserver  │
                      └──────────────────┬─────────────────────┘
                                         │
                                         ▼
                      ┌────────────────────────────────────────┐
                      │    IntegrationObserverTrait::syncIds() │
                      └──────────────────┬─────────────────────┘
                                         │
                                         ▼ (Dispatched to Queue Worker)
                      ┌────────────────────────────────────────┐
                      │      GlobalIntegrationSyncJob          │
                      │         (implements ShouldQueue)       │
                      └──────────────────┬─────────────────────┘
                                         │
                                         ▼
                      ┌────────────────────────────────────────┐
                      │  SalesforceService::syncBatch()        │
                      │   - Batch items by User & Integration  │
                      │   - Resolves Parent IDs (CrmLog)       │
                      │   - Calls Salesforce REST API v60.0    │
                      └────────────────────────────────────────┘
```

### Audit Trail & State Management (`CrmLog`)

Every asynchronous sync operation records its outcome in the database:

- **Table**: `bm_crm_logs`
- **Fields**: `provider` (`salesforce`), `model_type` (`contact`, `event`, `meeting`, `contact_note`), `model_id`, `parent_record_id` (Salesforce ID), `sync_status` (`success`, `failed`), and error details.
- Prevents redundant syncs and resolves parent-child relationships (e.g. attaching Notes to the correct Salesforce Contact ID).

---

## 7. Salesforce Flow Automation Triggers

Because BoothMaven maps each event capture to native Salesforce objects, RevOps and Admins can build automated Salesforce Flows without custom code:

1. **New Contact Created** &rarr; Trigger: `Contact.LeadSource = 'boothmaven'` &rarr; Action: Create Opportunity, assign territory rep, alert via Chatter.
2. **Hot Lead Score (Score &ge; 80)** &rarr; Trigger: `BM_Lead_Score__c >= 80` &rarr; Action: Immediately route to Senior AE and create high-priority follow-up Task within 2 hours.
3. **Qualifying Budget Approved** &rarr; Trigger: `BM_Qualification_Data__c` contains "Budget approved: Yes" &rarr; Action: Create Opportunity in "Evaluating" stage.
4. **Content Signal Task Created** &rarr; Trigger: `Task.Subject` starts with `📄 Content Signal:` &rarr; Action: Notify account owner via Slack/email that lead is reviewing collateral post-event.
5. **Two-Way Meeting Completed** &rarr; Trigger: `Event.ShowAs = 'Busy'` &rarr; Action: Advance Opportunity stage to "Proposal" and push completion outcome back to BoothMaven.
