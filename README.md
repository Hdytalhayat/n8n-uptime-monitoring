# Automated Website Uptime Monitor with State Management

An automated website and server uptime monitoring system built with **n8n, Docker, Google Sheets, and Discord Webhooks**. This project implements state management to detect downtime and recovery events while preventing duplicate notifications and unnecessary alert spam.

---

## Tech Stack

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=for-the-badge&logo=googlesheets&logoColor=white)
![Discord](https://img.shields.io/badge/Discord-5865F2?style=for-the-badge&logo=discord&logoColor=white)

---

## Background

The office website or server occasionally experiences downtime, particularly outside working hours. Unfortunately, the IT team often discovers these issues only after receiving complaints from clients or management.

This creates a need for an automated monitoring system that can detect service interruptions and immediately notify the IT team without requiring manual checks.

---

## Overview

Monitoring website availability should not overwhelm IT teams with unnecessary notifications.

Many basic monitoring systems send repeated alerts every time they check a website that is already down. This can lead to **alert fatigue**, making it harder for IT teams to identify important incidents.

This project addresses the issue by implementing **State Management** using Google Sheets as a lightweight state store.

Instead of sending notifications on every monitoring cycle, the workflow only sends alerts when a state transition occurs:

1. **Incident Detection (UP → DOWN):** Sends a notification when the website becomes unavailable.
2. **Service Recovery (DOWN → UP):** Sends a notification when the website becomes accessible again.

This approach prevents duplicate alerts while keeping the IT team informed about important service status changes.

---

## Workflow Architecture

The automation workflow is built with n8n and integrates HTTP health checks, Google Sheets, and Discord Webhooks.

![n8n Workflow](assets/n8n-canvas.png)

### Discord Incident & Recovery Notifications

The system sends different notifications depending on the website's status.

| Incident Alert (Down) | Recovery Alert (Resolved) |
|:---:|:---:|
| ![Down Alert](assets/discord-alert-down.png) | ![Resolved Alert](assets/discord-alert-resolved.png) |
| **Service Unavailable** | **Service Recovered** |

---

## How It Works (State Management Logic)

The workflow automatically checks the target website at a configurable interval (e.g., every 1–5 minutes).

It compares the current HTTP response with the previous status stored in Google Sheets to determine whether a notification should be sent.

### State Transition Matrix

| Previous State | Current HTTP Check | Action | Discord Notification |
|---|---|---|---|
| `AMAN` | HTTP ≠ 200 (Down) | Update state to `TIDAK AMAN` | 🔴 Send Incident Alert |
| `TIDAK AMAN` | HTTP ≠ 200 (Down) | No changes | Muted (Prevent Duplicate Alerts) |
| `TIDAK AMAN` | HTTP = 200 (Up) | Update state to `AMAN` | 🟢 Send Recovery Alert |
| `AMAN` | HTTP = 200 (Up) | No changes | Muted (Normal Operation) |

### Monitoring Flow

1. **Scheduled Trigger:** Executes the workflow at the configured interval.
2. **HTTP Health Check:** Sends an HTTP request to the target website or API endpoint.
3. **State Retrieval:** Retrieves the previous website status from Google Sheets.
4. **Status Comparison:** Compares the current HTTP response with the previously stored state.
5. **State Update:** Updates the Google Sheets record when a status transition occurs.
6. **Discord Notification:** Sends an incident or recovery notification when a state change is detected.

---

## Prerequisites & Tools

- [Docker](https://docs.docker.com/get-docker/) & Docker Compose
- [n8n](https://n8n.io/) (Self-hosted instance)
- Google Account (Google Sheets API access)
- Discord Server (with permission to create webhooks)
- A target website URL or REST API endpoint to monitor

---

## Installation Guide (Self-Hosted via Docker)

### 1. Clone the Repository

```bash
git clone https://github.com/Hdytalhayat/n8n-uptime-monitoring.git
cd n8n-uptime-monitoring
```

### 2. Configure Environment Variables

If the repository includes an environment template, copy it:

```bash
cp .env.example .env
```

Adjust the configuration parameters as needed, especially when deploying to a public VPS.

### 3. Start Docker Containers

```bash
docker compose up -d
```

Access your n8n instance at:

`http://localhost:5678`

---

## ⚙️ n8n Workflow Setup

1. Open your n8n dashboard and create an owner account if this is your first time running n8n.
2. Navigate to **Workflows** → Click the three-dot menu (`...`) → Select **Import from File**.
3. Import the workflow file: `workflows/uptime-monitor.json`.
4. Configure the required credentials and parameters:

   - **HTTP Request:** Enter the target website URL or REST API endpoint.
   - **Google Sheets:** Connect your Google account using OAuth2.
   - **Discord Webhook:** Replace the existing webhook URL with your own Discord Webhook URL.

5. Prepare your Google Spreadsheet using the data schema below.
6. Configure the monitoring interval according to your requirements.
7. Activate the workflow by changing its status from **Inactive** to **Active**.

---

## 📋 Google Sheets Data Schema

Create a Google Spreadsheet with the following columns to ensure compatibility with the workflow:

| Column | Description |
|---|---|
| `status_terakhir` | Stores the latest website status (`AMAN` or `TIDAK AMAN`). |
| `waktu_update` | Records the timestamp of the latest status update. |
| `row_number` | Stores the spreadsheet row reference used by the workflow. |

Make sure the initial status is set to `AMAN` if the website is currently accessible.

---

## Author

**Hidayatul Hayat**

- [LinkedIn](https://www.linkedin.com/in/hidayatul-hayat-s-kom-441b6b278)
- [Portfolio](https://portfolio-backend-frontend-murex.vercel.app/)
