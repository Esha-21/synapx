# SynapX – Autonomous Insurance Claims Processing Agent

> An AI-powered FNOL (First Notice of Loss) processing system that extracts fields, detects missing data, flags fraud, and routes claims automatically using Google Gemini AI.

---

## Overview

This agent solves the full FNOL processing pipeline:
- Extracts 16+ structured fields from unstructured claim documents
- Identifies missing or incomplete mandatory fields
- Classifies the claim type and applies routing rules
- Returns a JSON response with routing decision and reasoning

---

## Architecture

```
FNOL Document (paste text / upload PDF or TXT)
        │
        ▼
┌───────────────────────┐
│   Field Extraction    │  ← Google Gemini AI (gemini-2.5-flash)
│   via LLM Prompt      │    Extracts 16+ structured fields
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│  Missing Field Check  │  Validates all mandatory fields
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│   Routing Engine      │  Rule-based classifier
│   (4 routes)          │  Priority-ordered logic
└──────────┬────────────┘
           │
           ▼
  JSON Output + Visual UI
```

---

## Routing Rules (Priority Order)

| Priority | Route | Condition |
|---|---|---|
| 1 | **Investigation Flag** | Description contains: `fraud`, `inconsistent`, `staged`, `fabricated`, `suspicious`, `falsified` |
| 2 | **Specialist Queue** | Claim type includes `injury` or `personal injury` |
| 3 | **Manual Review** | Any mandatory field is missing or null |
| 4 | **Fast-track** | Estimated damage < ₹25,000 AND all fields present |
| 5 | **Manual Review** | Estimated damage ≥ ₹25,000 (standard processing) |

---

## Fields Extracted

| Category | Fields |
|---|---|
| Policy | Policy Number, Policyholder Name, Effective Date Start, Effective Date End |
| Incident | Date, Time, Location, Description |
| Parties | Claimant Name, Third Parties, Contact Email, Contact Phone |
| Asset | Asset Type, Asset ID, Estimated Damage |
| Claim | Claim Type, Attachments, Initial Estimate |

---

## Output Format

Every claim produces this JSON (matching the assignment spec):

```json
{
  "extractedFields": {
    "policyNumber": "POL-2024-78923",
    "policyholderName": "Rajesh Kumar",
    "effectiveDateStart": "01/04/2024",
    "effectiveDateEnd": "31/03/2025",
    "incidentDate": "12/11/2024",
    "incidentTime": "14:35",
    "incidentLocation": "NH-48, Pune-Mumbai Expressway",
    "incidentDescription": "Vehicle rear-ended by truck at low speed...",
    "claimantName": "Rajesh Kumar",
    "contactEmail": "rajesh.kumar@email.com",
    "contactPhone": "+91-98200-45678",
    "thirdParties": ["Suresh Patil - MH-12-BZ-4521"],
    "assetType": "Motor Vehicle",
    "assetId": "MH-01-CQ-9988",
    "estimatedDamage": 18500,
    "claimType": "Property Damage",
    "attachments": ["Photos of vehicle damage", "FIR copy"],
    "initialEstimate": 18500
  },
  "missingFields": [],
  "recommendedRoute": "Fast-track",
  "reasoning": "All mandatory fields are present. Estimated damage of 18500 is below the 25000 threshold. No fraud indicators detected. Claim qualifies for fast-track processing."
}
```

---

## Setup & Run

### Prerequisites

- [Node.js](https://nodejs.org) LTS version (v18 or above)
- A free Google Gemini API key

### Step 1 — Get a Free Gemini API Key

1. Go to https://aistudio.google.com/apikey
2. Sign in with your Google account
3. Click **"Create API Key"**
4. Copy the key (starts with `AIzaSy...`)

Free tier: 10 requests/min, 250 requests/day — no credit card required.

### Step 2 — Run the App

```bash
# Go into the project folder
cd synapx-claims-agent

# Start the local server (no npm install needed)
node server.js
```

You will see:
```
✅ SynapX Claims Agent is running!
👉 Open this in your browser: http://localhost:3000
```

### Step 3 — Open in Browser

Go to **http://localhost:3000**

Paste your Gemini API key into the yellow bar at the top, select a sample or paste your own FNOL text, and click **Analyze Claim**.

---

## Project Structure

```
synapx-claims-agent/
├── index.html        # Full frontend application (single file)
├── server.js         # Lightweight Node.js static file server
├── start.bat         # Windows one-click start script
├── HOW_TO_RUN.txt    # Quick start instructions
└── README.md
```

---

## Sample Scenarios (Built into the App)

| Sample | Scenario | Expected Route |
|---|---|---|
| Fast-track Auto Claim | Minor rear-end, 18500 damage | Fast-track |
| Missing Fields | Home flood, no contact/dates | Manual Review |
| Fraud Investigation Flag | Staged collision, inconsistent witnesses | Investigation Flag |
| Personal Injury to Specialist | Slip and fall with fractures | Specialist Queue |
| Complex Multi-Party Claim | 3-vehicle pile-up + injury | Specialist Queue |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript (no framework, no build step) |
| AI Model | Google Gemini API (gemini-2.5-flash) |
| Server | Node.js built-in http module |
| Fonts | Google Fonts — Syne, DM Mono |
| Icons | Tabler Icons |

---

## Approach & Design Decisions

**1. LLM for extraction, rules for routing**
Field extraction uses an LLM prompt rather than regex, making it robust to varied document formats, inconsistent spacing, and OCR artifacts. Routing logic is fully deterministic and rule-based — auditable and explainable.

**2. Priority-ordered routing**
Fraud flags override all other conditions. A low-damage claim with fraud keywords still goes to Investigation, not Fast-track. This mirrors real-world claims triage logic.

**3. Graceful handling of missing data**
Missing fields are listed explicitly in missingFields[] rather than silently ignored. Any missing mandatory field triggers Manual Review automatically.

**4. Risk scoring**
Each claim receives a 1–10 risk score alongside the route, giving adjusters a quick signal even within the same route category.

**5. Single-file frontend**
The entire UI is one index.html file — no npm, no bundler, no framework. Easy to inspect, modify, and deploy anywhere.

---

## Author

Built for the SynapX Assessment — Autonomous Insurance Claims Processing Agent.
