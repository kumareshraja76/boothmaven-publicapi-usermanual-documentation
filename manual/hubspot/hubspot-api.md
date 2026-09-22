---
title: "HubSpot Integration API"
sidebarTitle: "HubSpot Integration"
description: "Learn how to connect BoothMaven with HubSpot and synchronize contacts and other supported data."
icon: "link"
---

# BoothMaven - HubSpot Integration API Documentation

> **Reference URL**: [https://www.boothmaven.com/integration/hubspot/](https://www.boothmaven.com/integration/hubspot/)  
> **API Version**: HubSpot CRM API v3 / v4 Associations  
> **Supported Plans**: **All Plans** — Capture ($49/mo), Essential ($149/mo), Business ($399/mo)  
> **Contact / Support**: [hello@boothmaven.com](mailto:hello@boothmaven.com) &bull; [BoothMaven.com](https://boothmaven.com)

---

## Overview

BoothMaven delivers real-time synchronization between on-site event lead capture and HubSpot CRM. Within seconds of a badge scan or business card exchange, full contact records, event metadata, qualifying answers, meeting schedules, and engagement signals land directly in HubSpot.

### Four Layers of Event Intelligence

1. **Contact & Company Intelligence**: Standard contact fields, company properties, and automated Apollo.io enrichment (industry, revenue, headcount, tech stack).
2. **Event Context & Qualification**: Event title, booth number, capturing representative, lead score (0–100), and custom qualifying survey responses mapped to named properties.
3. **Meeting Activities & Conversation Notes**: Scheduled booth meetings, attendee responses (Accepted/Declined), and voice memo audio transcriptions synced as native HubSpot Notes and Meetings.
4. **Content Engagement Signals (Timeline Activities)**: Post-event document opens, video views, time spent reading, and multi-touch revisit signals that trigger automated HubSpot Workflows.

---

## Table of Contents

1. [Authentication & Connection (OAuth 2.0)](#1-authentication--connection-oauth-20)
2. [Data Model & Object Architecture](#2-data-model--object-architecture)
3. [API Endpoints & Payloads](#3-api-endpoints--payloads)
   - [3.1 Contact Sync (Create & Update)](#31-contact-sync-create--update)
   - [3.2 Custom Event Objects (App Object Schema)](#32-custom-event-objects-app-object-schema)
   - [3.3 Meeting Activities & Attendees](#33-meeting-activities--attendees)
   - [3.4 Conversation Notes (Voice & Text)](#34-conversation-notes-voice--text)
   - [3.5 Content Engagement Signals (Timeline Activities)](#35-content-engagement-signals-timeline-activities)
4. [Field Mapping Specifications](#4-field-mapping-specifications)
5. [Deduplication & Auto-Healing](#5-deduplication--auto-healing)
6. [Asynchronous Trigger Points & Execution Architecture](#6-asynchronous-trigger-points--execution-architecture)
   - [Sync vs. Async Trigger Matrix](#sync-vs-async-trigger-matrix)
   - [Job Dispatch Pipeline & Observers](#job-dispatch-pipeline--observers)
   - [Cascade Synchronization](#cascade-synchronization)
   - [State Tracking (CrmLog)](#state-tracking-crmlog)
7. [HubSpot Workflow Automation Triggers](#7-hubspot-workflow-automation-triggers)

---

## 1. Authentication & Connection (OAuth 2.0)

BoothMaven connects to HubSpot portals using the official **HubSpot App OAuth 2.0** flow.

### OAuth Endpoints

| Step               | Method | URL                                       | Description                                         |
| :----------------- | :----- | :---------------------------------------- | :-------------------------------------------------- |
| **Authorize**      | `GET`  | `https://app.hubspot.com/oauth/authorize` | Initiates portal selection and OAuth consent screen |
| **Token Exchange** | `POST` | `https://api.hubapi.com/oauth/v1/token`   | Exchanges authorization code for tokens             |
| **Token Refresh**  | `POST` | `https://api.hubapi.com/oauth/v1/token`   | Refreshes expired access tokens                     |

#### Required OAuth Scopes

- `crm.objects.contacts.write` & `crm.objects.contacts.read`: Contact creation and updates.
- `crm.objects.companies.write` & `crm.objects.companies.read`: Company associations.
- `crm.objects.custom.read` & `crm.objects.custom.write`: BoothMaven Event App Objects.
- `timeline`: Publishing Content Signal Timeline Events.
- `crm.schemas.contacts.read`: Schema and custom property discovery.

#### Token Refresh Payload (`POST https://api.hubapi.com/oauth/v1/token`)

```http
POST /oauth/v1/token HTTP/1.1
Host: api.hubapi.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&client_id=YOUR_CLIENT_ID
&client_secret=YOUR_CLIENT_SECRET
&refresh_token=YOUR_REFRESH_TOKEN
```

---

## 2. Data Model & Object Architecture

```
┌────────────────────────────────────────────────────────┐
│                   HubSpot Contact                      │
│  (Email, Phone, Lead Score, Apollo Enriched Company)   │
└───────────────────────────┬────────────────────────────┘
                            │
         ┌──────────────────┼──────────────────┐
         ▼                  ▼                  ▼
┌──────────────────┐┌──────────────────┐┌──────────────────┐
│   HubSpot Note   ││ HubSpot Meeting  ││ Custom App Event │
│ (Voice Memo Text ││(Scheduled Booth  ││(Event Metadata,  │
│  & Transcripts)  ││ Demos & Outcomes)││ Booth #, Rep)    │
└──────────────────┘└──────────────────┘└──────────────────┘
                            │
                            ▼
               ┌──────────────────────────┐
               │    Timeline Activity     │
               │(Content Signals: Pricing │
               │ Sheet, Video Views, etc.)│
               └──────────────────────────┘
```

---

## 3. API Endpoints & Payloads

### 3.1 Contact Sync (Create & Update)

Creates a new Contact or updates an existing Contact matched by email address.

- **Create Endpoint**: `POST https://api.hubapi.com/crm/v3/objects/contacts`
- **Update Endpoint**: `PATCH https://api.hubapi.com/crm/v3/objects/contacts/{contactId}`

#### Request Payload (`SimplePublicObjectInput`)

```json
{
  "properties": {
    "email": "sarah.chen@techscale.com",
    "firstname": "Sarah",
    "lastname": "Chen",
    "phone": "+1-650-555-0144",
    "mobilephone": "+1-650-555-0188",
    "company": "TechScale Inc",
    "jobtitle": "VP Marketing",
    "city": "San Francisco",
    "country": "United States",
    "industry": "Software & Internet",
    "website": "https://techscale.com",
    "hs_lead_status": "CONNECTED",
    "bm_social_links": "LinkedIn: https://linkedin.com/in/sarahchen\nTwitter: https://twitter.com/sarahchen",
    "bm_qualification_data": "Budget approved: Yes — Q3 2026\nDecision authority: Final say\nTimeline: 6–9 months\nCurrent Vendor: Legacy Supplier"
  }
}
```

---

### 3.2 Custom Event Objects (App Object Schema)

BoothMaven creates custom event records under its dedicated HubSpot App Object prefix (`a44678917_`).

- **Endpoint**: `POST https://api.hubapi.com/crm/v3/objects/2-12345678` _(Custom Event Object Type ID)_

#### Request Payload

```json
{
  "properties": {
    "a44678917_event_id": "104",
    "a44678917_event_title": "Dreamforce 2026",
    "a44678917_description": "Annual Global Cloud & AI Expo",
    "a44678917_start_date_and_time": "2026-09-14",
    "a44678917_end_date_and_time": "2026-09-17",
    "a44678917_venue": "Moscone Center",
    "a44678917_location": "San Francisco, CA",
    "a44678917_booth_number": "#2248",
    "a44678917_event_status": "in_progress",
    "a44678917_attendance_format": "in_person"
  }
}
```

---

### 3.3 Meeting Activities & Attendees

Booth bookings sync as HubSpot Meetings. Attendee responses (Accepted/Declined) create associated status notes.

- **Create Meeting**: `POST https://api.hubapi.com/crm/v3/objects/meetings`
- **Associate to Contact (v4)**: `PUT https://api.hubapi.com/crm/v4/objects/meetings/{meetingId}/associations/contacts/{contactId}`

#### Meeting Request Payload

```json
{
  "properties": {
    "hs_timestamp": 1789482600000,
    "hs_meeting_title": "Product Demo · Dreamforce 2026",
    "hs_meeting_body": "Deep dive into compliance automation and multi-booth lead tracking.",
    "hs_meeting_location": "Booth #2248 Demo Pod B",
    "hs_meeting_location_type": "CUSTOM",
    "hs_meeting_start_time": 1789482600000,
    "hs_meeting_end_time": 1789484400000,
    "hs_meeting_outcome": "SCHEDULED",
    "hubspot_owner_id": "12345678"
  }
}
```

#### Meeting Association Definition (v4)

```json
[
  {
    "associationCategory": "HUBSPOT_DEFINED",
    "associationTypeId": 200
  }
]
```

---

### 3.4 Conversation Notes (Voice & Text)

Transcribed booth conversations and audio memo notes are created and associated directly with the Contact:

- **Endpoint**: `POST https://api.hubapi.com/crm/v3/objects/notes`
- **Association**: Linked to Contact via `associationTypeId: 202`

#### Note Request Payload

```json
{
  "properties": {
    "hs_note_body": "<b>Booth Conversation Note (Voice Memo Transcribed):</b><br>Evaluating Q3. Current vendor has reliability issues. Budget approved. Wants factory demo. Follow up Thursday.",
    "hs_timestamp": "2026-09-14T14:35:00.000Z"
  }
}
```

---

### 3.5 Content Engagement Signals (Timeline Activities)

When an attendee engages with digital cards or downloaded content post-event, a Timeline Activity is posted to the contact's record.

#### Timeline Event Payload

```json
{
  "eventTemplateId": "bm_content_signal_v1",
  "objectId": "1001",
  "tokens": {
    "document_title": "Pricing Sheet 2026",
    "duration_formatted": "4 minutes 12 seconds",
    "visit_number": "2nd visit",
    "channel": "Email link (Quick Share)",
    "viewed_at": "2026-09-17T11:42:00Z"
  }
}
```

---

## 4. Field Mapping Specifications

| BoothMaven Field        | HubSpot Property Name           | Property Type     | Plan Tier  |
| :---------------------- | :------------------------------ | :---------------- | :--------- |
| First Name              | `firstname`                     | String            | Capture+   |
| Last Name               | `lastname`                      | String            | Capture+   |
| Email Address           | `email`                         | String (Identity) | Capture+   |
| Phone Number            | `phone`                         | String            | Capture+   |
| Mobile Phone            | `mobilephone`                   | String            | Capture+   |
| Job Title / Designation | `jobtitle`                      | String            | Capture+   |
| Company Name            | `company`                       | String            | Capture+   |
| City                    | `city`                          | String            | Capture+   |
| Country                 | `country`                       | String            | Capture+   |
| Industry                | `industry`                      | String            | Essential+ |
| Website                 | `website`                       | String            | Capture+   |
| Lead Status             | `hs_lead_status`                | String            | Capture+   |
| Social Profiles         | `bm_social_links`               | Multi-line text   | Capture+   |
| Qualifying Answers      | `bm_qualification_data`         | Multi-line text   | Essential+ |
| Lead Score (0–100)      | `bm_lead_score`                 | Number            | Essential+ |
| Event Title             | `a44678917_event_title`         | Custom App Object | Capture+   |
| Booth Number            | `a44678917_booth_number`        | Custom App Object | Capture+   |
| Event Dates             | `a44678917_start_date_and_time` | Date              | Capture+   |
| Meeting Title           | `hs_meeting_title`              | Meeting Object    | Essential+ |
| Meeting Outcome         | `hs_meeting_outcome`            | Meeting Object    | Essential+ |
| Content Signals         | Timeline Activity               | Timeline Event    | Essential+ |

---

## 5. Deduplication & Auto-Healing

### 1. 409 Conflict Deduplication Resolution

If HubSpot returns an HTTP `409 Conflict` (indicating a duplicate contact exists by email):

1. BoothMaven parses the existing contact ID from the response message: `Existing ID: (\d+)`.
2. Automatically falls back to a `PATCH` update against that discovered ID.
3. Continues execution smoothly without dropping lead data.

### 2. Auto-Healing Missing Custom Properties

If a user's HubSpot portal lacks required custom properties (e.g. `bm_qualification_data` or `bm_social_links`):

1. BoothMaven catches the HTTP `400 Bad Request` schema error.
2. Calls `ensureCustomContactProperties()` to provision the missing properties via HubSpot's Schemas API.
3. Automatically retries the contact sync request transparently.

---

## 6. Asynchronous Trigger Points & Execution Architecture

To ensure zero latency on mobile badge scanning and immediate lead capture offline or online, **all HubSpot sync actions execute asynchronously** through Laravel Queue workers.

### Sync vs. Async Trigger Matrix

| User / System Action             | Trigger Point                   | Execution Mode           | Queue Job / Handler                                          |
| :------------------------------- | :------------------------------ | :----------------------- | :----------------------------------------------------------- |
| **OAuth Connect**                | User clicks Connect in Settings | **Synchronous**          | `HubSpotOAuthController@callback`                            |
| **Connection Test**              | User clicks "Test Connection"   | **Synchronous**          | `HubSpotService@testConnection`                              |
| **Token Refresh**                | Access token expiry             | **Synchronous** (Cached) | `HubSpotService@getAccessToken`                              |
| **Lead Captured at Booth**       | Scanner saves contact           | **⚡ Asynchronous**      | `CustomerContactsObserver` &rarr; `GlobalIntegrationSyncJob` |
| **Event Created / Edited**       | Event created in portal         | **⚡ Asynchronous**      | `EventObserver` &rarr; `GlobalIntegrationSyncJob`            |
| **Attendee Event Check-in**      | Lead checked into event         | **⚡ Asynchronous**      | `ContactEventObserver` &rarr; `GlobalIntegrationSyncJob`     |
| **Meeting Scheduled**            | Booth demo booked               | **⚡ Asynchronous**      | `ContactMeetingObserver` &rarr; `GlobalIntegrationSyncJob`   |
| **Meeting Outcome Updated**      | Outcome updated (Done/NoShow)   | **⚡ Asynchronous**      | `MeetingObserver` &rarr; `GlobalIntegrationSyncJob`          |
| **Voice / Text Note Added**      | Voice memo transcribed          | **⚡ Asynchronous**      | `ContactNoteObserver` &rarr; `GlobalIntegrationSyncJob`      |
| **Meeting Note Added**           | Minutes added to meeting        | **⚡ Asynchronous**      | `MeetingNoteObserver` &rarr; `GlobalIntegrationSyncJob`      |
| **Lead Score Recalculated**      | Signals update score            | **⚡ Asynchronous**      | `LeadScoringEngine` &rarr; `GlobalIntegrationSyncJob`        |
| **Apollo.io Company Enrichment** | Domain enrichment ready         | **⚡ Asynchronous**      | `ProcessEnrichmentJob` &rarr; `GlobalIntegrationSyncJob`     |

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
                      │    HubSpotService::syncBatch()         │
                      │   - Resolves Existing IDs (CrmLog)     │
                      │   - Calls HubSpot CRM v3 APIs          │
                      │   - Executes cascadeSyncContactRelations│
                      └────────────────────────────────────────┘
```

### Cascade Synchronization

When a contact is newly synced to HubSpot, `cascadeSyncContactRelations()` immediately syncs related event attendance, scheduled meetings, and notes for that contact, eliminating race conditions between related records.

### State Tracking (`CrmLog`)

Every asynchronous sync operation is logged in the `bm_crm_logs` table with provider `hubspot`, recording the external HubSpot ID, timestamp, and status.

---

## 7. HubSpot Workflow Automation Triggers

Each event captured by BoothMaven can trigger native HubSpot Workflows:

1. **New Contact Captured** &rarr; Enrol in post-event nurturing sequence and send instant Slack notification to the booth team.
2. **Hot Lead Score (&ge; 80)** &rarr; Create high-priority AE task with 2-hour SLA and alert the Sales Director.
3. **Pricing Sheet Viewed (Content Signal)** &rarr; Trigger: Timeline activity `Pricing Sheet Viewed` &rarr; Action: Create AE task to follow up within 4 hours.
4. **Meeting Completed** &rarr; Advance deal stage to "Demo Delivered" and trigger proposal request sequence.
5. **Meeting No-Show** &rarr; Trigger: Meeting outcome = `NO_SHOW` &rarr; Action: Enrol in rescheduling sequence with self-booking link.
6. **"Met Again" Visitor** &rarr; Contact captured at a 2nd trade show &rarr; Flag as key account and notify account executive of multi-event interest.
