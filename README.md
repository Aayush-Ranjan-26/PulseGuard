# 🛡️ PulseGuard: Automated Infrastructure Health Sentinel & Incident Response Pipeline 🚀🎉

<p align="center">
  <img src="https://img.shields.io/badge/n8n-Automated_Workflow-EA4B71?style=for-the-badge&logo=n8n&logoColor=white" alt="n8n" />
  <img src="https://img.shields.io/badge/Monitoring-Real--Time_Health_Check-4CAF50?style=for-the-badge&logo=prometheus&logoColor=white" alt="Monitoring" />
  <img src="https://img.shields.io/badge/Discord-Instant_Incident_Alerts-5865F2?style=for-the-badge&logo=discord&logoColor=white" alt="Discord" />
  <img src="https://img.shields.io/badge/Google_Sheets-Audit_Trail_&_Metrics-34A853?style=for-the-badge&logo=googlesheets&logoColor=white" alt="Google Sheets" />
  <img src="https://img.shields.io/badge/Security-Zero_Hardcoded_Secrets-blue?style=for-the-badge&logo=shield&logoColor=white" alt="Security" />
  <img src="https://img.shields.io/badge/Status-Production--Ready-success?style=for-the-badge" alt="Status" />
</p>

---

## 📘 Executive Overview & Highlights

**PulseGuard** is an enterprise-grade, fully automated infrastructure heartbeat and telemetry orchestration pipeline engineered within **n8n**. Designed to deliver relentless visibility over homelabs, web applications, microservices, and network endpoints, PulseGuard operates continuously in the background—evaluating response status codes, profiling round-trip latency, streaming structured operational logs to cloud telemetry sheets, and triggering high-priority incident notifications across modern messaging channels the instant an anomaly is detected! ⚡🌐

### 🌟 Why PulseGuard?
* 🚀 **Autonomous 24/7 Monitoring**: Executes deterministic cron triggers every 5 minutes with zero human intervention required.
* ⏱️ **Precision Latency Profiling**: Captures millisecond-accurate HTTP round-trip timing (`response_time_ms`) alongside standard RFC status codes.
* 🚨 **Intelligent Incident Triaging**: Instantly differentiates between operational uptime, network timeouts, DNS failures, and performance degradation (> 2000 ms).
* 🔔 **Instant Discord Webhook Dispatch**: Issues rich, human-readable triage cards to system administrators the exact moment a service stumbles.
* 📊 **Cloud Audit Trail & Analytics**: Streams perpetual timeseries performance metrics into Google Sheets, ready for visualization, SLA calculation, and uptime graphing.
* 🔒 **Privacy-First & Secure by Design**: Pre-sanitized template featuring zero hardcoded secrets, isolated environment variable fallback support, and clean credential placeholders! 🎉

---

## 📑 Table of Contents

1. [🏛️ Architecture & Incident Data Flow](#️-architecture--incident-data-flow)
2. [🧩 Workflow Node Breakdown](#-workflow-node-breakdown)
3. [⚙️ Prerequisites & System Requirements](#️-prerequisites--system-requirements)
4. [📘 Placeholder & Configuration Guide](#-placeholder--configuration-guide)
   - [1. Discord Webhook Configuration](#1-discord-webhook-configuration)
   - [2. Google Spreadsheet ID Configuration](#2-google-spreadsheet-id-configuration)
   - [3. Google Sheets OAuth2 Credential Linking](#3-google-sheets-oauth2-credential-linking)
   - [4. Defining Monitored Endpoints](#4-defining-monitored-endpoints)
5. [🚀 Step-by-Step Installation & Deployment](#-step-by-step-installation--deployment)
6. [🧪 Testing & Incident Simulation](#-testing--incident-simulation)
7. [🛡️ Incident Thresholds & Logic Rules](#️-incident-thresholds--logic-rules)
8. [💡 Customization & Scaling Horizons](#-customization--scaling-horizons)

---

## 🏛️ Architecture & Incident Data Flow

```
                      ┌───────────────────────────┐
                      │  ⏰ Cron - Every 5 Min    │
                      │  (Deterministic Trigger)  │
                      └─────────────┬─────────────┘
                                    │
                                    ▼
                      ┌───────────────────────────┐
                      │  📋 Define Services       │
                      │  (Endpoints & Metadata)   │
                      └─────────────┬─────────────┘
                                    │
                                    ▼
                      ┌───────────────────────────┐
                      │  🔀 Split Services        │
                      │  (Individual Item Stream) │
                      └─────────────┬─────────────┘
                                    │
                                    ▼
                      ┌───────────────────────────┐
                      │  ⚡ Check Service          │
                      │  • HTTP Ping (10s Timeout)│
                      │  • Latency Calculation    │
                      │  • Error & DNS Trapping   │
                      └─────────────┬─────────────┘
                                    │
                                    ▼
                      ┌───────────────────────────┐
                      │  ⚖️ IF Down or Slow       │
                      │  • status_code != 200     │
                      │         -- OR --          │
                      │  • response_time_ms > 2000│
                      └──────┬─────────────┬──────┘
                             │             │
                    [ TRUE / INCIDENT ]    [ FALSE / NOMINAL ]
                             │             │
                             ▼             ▼
  ┌────────────────────────────┐         ┌────────────────────────────┐
  │ 🚨 Discord Alert           │         │ 📊 Log to Sheets (OK)      │
  │ (Rich Webhook Notification)│         │ (Appends Healthy Telemetry)│
  └──────────────┬─────────────┘         └────────────────────────────┘
                 │
                 ▼
  ┌────────────────────────────┐
  │ 📊 Log to Sheets (Alert)   │
  │ (Appends Outage Incident)  │
  └────────────────────────────┘
```

---

## 🧩 Workflow Node Breakdown

| Node Name | Node Type | Purpose & Technical Execution |
| :--- | :--- | :--- |
| **⏰ Cron - Every 5 Min** | `n8n-nodes-base.scheduleTrigger` | Generates a scheduled pulse every 300 seconds to initiate the diagnostic evaluation cycle. |
| **📋 Define Services** | `n8n-nodes-base.set` | Houses the registry of target services, mapping friendly service names to their destination URLs. |
| **🔀 Split Services** | `n8n-nodes-base.splitOut` | De-aggregates the service array into independent item records for downstream parallel processing. |
| **⚡ Check Service (HTTP + Timing)** | `n8n-nodes-base.code` | Executes an asynchronous HTTP ping with a strict 10-second timeout, measuring delta timestamp (`ms`), capturing status codes, and shielding against fatal DNS or connection crashes. |
| **⚖️ IF Down or Slow** | `n8n-nodes-base.if` | Evaluates dual operational conditions: evaluates whether `status_code != 200` OR `response_time_ms > 2000`. Routes traffic accordingly. |
| **🚨 Discord Alert** | `n8n-nodes-base.httpRequest` | Crafts an immediate incident dispatch to your designated Discord channel with emoji-coded diagnostic metrics. |
| **📊 Log to Sheets (Alert)** | `n8n-nodes-base.googleSheets` | Appends incident-state records into your central Google Sheet for SLA breach tracking and root-cause analysis. |
| **📊 Log to Sheets (OK)** | `n8n-nodes-base.googleSheets` | Appends healthy baseline telemetry into your Google Sheet to maintain complete historical health curves. |

---

## ⚙️ Prerequisites & System Requirements

Before deploying the PulseGuard pipeline, ensure the following tools and services are accessible:

* **n8n Host Environment**:
  - n8n `v1.x` or higher (compatible with Docker, self-hosted npm, or n8n Cloud).
  - Node.js `v18.x` or `v20.x` (if hosting via npm).
* **Discord Community or Server**:
  - Administrative permission to create an incoming webhook on any text channel.
* **Google Cloud & Google Sheets**:
  - A Google account with access to [Google Sheets](https://sheets.google.com).
  - Google Cloud project with the **Google Sheets API** enabled and OAuth2 credentials configured in n8n.

---

## 📘 Placeholder & Configuration Guide

To ensure pristine credential safety, this repository utilizes standard placeholders and dynamic expression bindings. Follow these steps prior to activating your workflow:

### 1. Discord Webhook Configuration 🚨
The **Discord Alert** node is pre-configured with a dynamic environment expression:
```javascript
={{ $env.DISCORD_WEBHOOK_URL || 'https://discord.com/api/webhooks/YOUR_WEBHOOK_ID/YOUR_WEBHOOK_TOKEN' }}
```
You can configure this via either method:
* **Option A (Environment Variable - Recommended for Production)**:
  Supply `DISCORD_WEBHOOK_URL` in your n8n `.env` file or container environment:
  ```env
  DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/123456789012345678/abcdefghijklmnopqrstuvwxyz_1234567890
  ```
* **Option B (Direct Node Editing)**:
  Open the **Discord Alert** node, locate the `URL` parameter, and paste your webhook URL directly over the placeholder.

### 2. Google Spreadsheet ID Configuration 📊
Both logging nodes (`Log to Sheets (Alert)` and `Log to Sheets (OK)`) contain the clean placeholder:
```text
YOUR_GOOGLE_SPREADSHEET_ID
```
1. Create a new Google Spreadsheet (e.g., `Infrastructure Telemetry Log`).
2. Copy the unique identifier located between `/d/` and `/edit` in your browser URL bar:
   ```text
   https://docs.google.com/spreadsheets/d/1A2B3C4D5E6F7G8H9I0J_EXAMPLE_SHEET_ID/edit
                                          ▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲
   ```
3. Open **Log to Sheets (Alert)** and **Log to Sheets (OK)**, switch the Document selector to **By ID**, and enter your copied ID.

### 3. Google Sheets OAuth2 Credential Linking 🔑
1. In your n8n workspace, navigate to **Credentials > Add Credential > Google Sheets OAuth2 API**.
2. Authenticate your authorized Google account.
3. Open both **Log to Sheets (Alert)** and **Log to Sheets (OK)** nodes, and select your authenticated credential from the dropdown menu.

### 4. Defining Monitored Endpoints 📋
Open the **Define Services** node. Customize the JSON array to include whatever servers, media centers, APIs, or endpoints you wish to track:
```json
[
  { "name": "Plex Media Server", "url": "https://plex.example.com/" },
  { "name": "TrueNAS Storage",   "url": "https://nas.example.local/" },
  { "name": "Home Assistant",    "url": "https://homeassistant.example.org/" },
  { "name": "Public API Gateway", "url": "https://api.example.com/health" }
]
```

---

## 🚀 Step-by-Step Installation & Deployment

### Step 1: Prepare the Google Sheet 📑
1. Open [Google Sheets](https://sheets.new) to create a new spreadsheet.
2. Rename the active sheet tab to `Sheet1`.
3. In **Row 1**, define the exact column headers below:
   | timestamp | service_name | status_code | response_time_ms |
   | :--- | :--- | :--- | :--- |
4. *(Optional Tip)*: Freeze Row 1 and style the header row with bold text and a subtle background for polished readability.

### Step 2: Import the Workflow into n8n 📥
1. Launch your n8n instance.
2. In the left navigation sidebar, click **Workflows**.
3. In the top-right corner, click **Add Workflow** > **Import from File...**
4. Select the [`N8N Lab Health Monitor.json`](./N8N%20Lab%20Health%20Monitor.json) file from this repository.

### Step 3: Connect Credentials & Placeholders 🔗
1. Connect your **Google Sheets OAuth2 API** credential to both logging nodes.
2. Insert your **Google Spreadsheet ID** into both logging nodes.
3. Configure your **Discord Webhook URL** in the alert node.
4. Click **Save** in the top navigation bar.

### Step 4: Turn Workflow Active 🎉
Toggle the workflow switch in the top-right corner from **Inactive** to **Active**. PulseGuard is now diligently watching over your infrastructure! 🛡️

---

## 🧪 Testing & Incident Simulation

To verify that alerts and telemetry stream as expected without waiting for an authentic failure:

1. Open the **Define Services** node and temporarily append an invalid or failing endpoint:
   ```json
   { "name": "Simulated Outage Server", "url": "https://httpstat.us/503" },
   { "name": "Simulated Latency Server", "url": "https://httpstat.us/200?sleep=3000" }
   ```
2. Click **Test Workflow** in the bottom center of the n8n canvas.
3. **Examine the Results**:
   * 🎉 Check your Discord channel: An immediate alert card formatted with status code `503` or latency `3000+ ms` will arrive instantly!
   * 📊 Check your Google Sheet: Look for freshly appended rows documenting the timestamp, service name, status code, and latency!

---

## 🛡️ Incident Thresholds & Logic Rules

PulseGuard relies on a resilient diagnostic JavaScript engine that intercepts every network state:

```javascript
// Strict 10-Second Timeout Buffer with Graceful Fallback
try {
  const res = await this.helpers.httpRequest({
    method: 'GET',
    url: service.url,
    timeout: 10000,
    returnFullResponse: true,
    ignoreHttpStatusErrors: true,
  });
  statusCode = res.statusCode;
} catch (e) {
  statusCode = 0; // Captures DNS resolution errors, connection drops, and socket timeouts
}
```

* **Healthy Nominal State (Route False)**:
  * `status_code === 200` **AND** `response_time_ms <= 2000`
* **Incident / Anomaly State (Route True)**:
  * `status_code !== 200` (e.g. `400`, `401`, `404`, `500`, `502`, `503`, or `0` for unresolvable servers)
  * **OR** `response_time_ms > 2000` (Degraded response latency exceeding 2.0 seconds)

---

## 💡 Customization & Scaling Horizons

* 📱 **Multi-Channel Dispatching**: Want alerts via **Telegram**, **Slack**, or **SMS (Twilio)**? Simply branch additional messaging nodes off the `True` port of **IF Down or Slow**.
* ⏱️ **Tuning Sampling Rates**: Adjust the interval inside **Cron - Every 5 Min** to 1 minute for mission-critical production systems or 15 minutes for lightweight personal labs.
* 📈 **Automated Dashboard Integration**: Connect your Google Sheet directly to **Looker Studio** or **Grafana** (via Google Sheets plugin) to generate live, real-time uptime percentage graphs and latency trends!

---

<p align="center">
  <b>Built with ❤️ and engineered for resilient infrastructure observability. Keep your systems up & your data protected! 🎉🚀</b>
</p>
