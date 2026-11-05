# AgapAI: An On-Demand Disaster Relief Coordination System

**Integrating Social Media Classification, Geoparsing, and Victim-Volunteer Matching**

A thesis project presented to the Faculty of the School of Computer Studies, University of San Jose–Recoletos, in partial fulfillment of the requirements for the degree of Bachelor of Science in Computer Science.

> **Status:** Work in progress. Retrieval, classification, spatiotemporal retrieval, event clustering, social network analysis and the dashboard are implemented. Geographic distance is not yet used in volunteer matching (a priority fix), and most system-level testing is still pending. See [Development Status](#development-status).

## Team

| Name | 
|---|
| Alessa Arong |
| Mary Vincent Lopez |
| Jeramie Anne Losorata |

**Adviser:** Ms. Josephine Petralba

## Overview

During disasters, many victims ask for help and many volunteers offer it on social media, but the posts are scattered, unstructured and hard to act on. After the magnitude 6.9 northern Cebu earthquake on September 30, 2025, help was not lacking; coordination was.

AgapAI is a web dashboard for disaster coordinators in Cebu Province, such as barangay officials or NGO volunteer leads. A coordinator picks a date range and clicks **Load Data**. AgapAI then:

1. Searches Bluesky for public posts about floods, earthquakes, typhoons, fires and landslides in Cebu.
2. Keeps only new posts that are really about Cebu.
3. Classifies each post's role, urgency, disaster type, intent and the resources needed or offered.
4. Finds the location and the event date and time each post describes.
5. Groups posts about the same incident into one event and pins it on a map.
6. For a selected victim, ranks volunteers who offer what the victim needs, using how they are connected on Bluesky.

AgapAI never acts on its own. It organizes the information, and the coordinator decides what to do.

### Gaps this project addresses

- **One tool, one step:** existing tools cover only collection, classification, manual geotagging or matching. AgapAI chains them in one map-based system.
- **Social connections are ignored:** no existing tool uses the follow relationships between posters as a matching signal.
- **Local language support:** most tools are built for English. AgapAI is built for English, Filipino, Cebuano and mixed-language posts, and for Cebu's barangay-level place names.

## Features

- On-demand retrieval of public Bluesky posts for a chosen date range (incremental, with duplicate and validity checks)
- Rule-based Cebu relevance filtering against the PSA PSGC gazetteer
- Multilingual multitask classifier (XLM-RoBERTa) for role, urgency, disaster type, intent and eight resource categories
- Spatiotemporal retrieval: locations from post text, author bio or earlier posts, and event date and time anchored to the posting time (Asia/Manila)
- Event clustering with DBSCAN (same disaster type, within 2 km and 6 hours)
- Social network analysis: volunteers ranked by matched resources, hop distance and Jaccard similarity
- Interactive dashboard with Posts, Users, Spatiotemporal Info, Map View and SNA tabs, plus direct messaging to a matched volunteer through Bluesky

## How It Works

The pipeline has seven phases. Phases 1 to 4 run on each **Load Data** click, Phase 5 groups the results into events, and Phase 6 runs when a coordinator selects a victim in the SNA tab.

| # | Phase | What it does |
|---|---|---|
| 1 | Bluesky data retrieval | Checks MongoDB first, then searches Bluesky with Cebu location and disaster terms through the AT Protocol, pages through results and removes duplicates and invalid posts |
| 2 | Cebu relevance filtering | Rule-based check against the PSGC gazetteer (53 cities and municipalities, 971 barangays, 23 landmarks), with a cascade that falls back to the author's bio and recent posts |
| 3 | Multitask classification | One XLM-RoBERTa encoder with six output heads: role, urgency, disaster type, intent, resources and resource count |
| 4 | Spatiotemporal retrieval | calamanCy location candidates validated against the gazetteer, author-context fallback, nearby-location disambiguation (5 km), and date and time parsing with translation fallback |
| 5 | Event clustering | Links posts of the same type within 2 km and 6 hours and groups them with DBSCAN |
| 6 | SNA and volunteer matching | Builds a NetworkX graph (up to 3 hops) and ranks volunteers by matched resources, then fewer hops, then higher Jaccard similarity |
| 7 | Dashboard and map | React, Leaflet and SVG views of posts, events and the social graph |

### Classifier labels

| Task | Labels |
|---|---|
| Role | Victim, Volunteer, General Info |
| Urgency | Very High, High, Medium, Low |
| Disaster type | Earthquake, Fire, Flood, Landslide, Typhoon, General disaster |
| Intent | Need, Offer, None |
| Resources (8) | Water, Food, Clothing and Bedding, Medicines/Medical Care, Shelter, Sanitation, Rescue, General Relief |

## Tech Stack

| Layer | Tools |
|---|---|
| Backend | Python 3.11, FastAPI, Uvicorn, Pydantic |
| Database | MongoDB 6.0 (PyMongo) |
| Social data | Bluesky AT Protocol (`atproto` SDK) |
| ML / NLP | PyTorch, Hugging Face Transformers, `xlm-roberta-base`, scikit-learn, pandas, NumPy |
| Geoparsing | spaCy with calamanCy (`ljvmiranda921/tl_calamancy_md`), PSGC gazetteer (1,257 records) |
| Date and time | Microsoft Recognizers-Text DateTime, dateparser, langdetect, NLLB-200 (`facebook/nllb-200-distilled-600M`) |
| Graph analysis | NetworkX |
| Frontend | React 19, Vite, Leaflet / react-leaflet, OpenStreetMap tiles, SVG |

All runtime tools are open-source or open-weight with no per-call fee, so the system does not depend on a paid outside API during a disaster.

## Model and Dataset

- **Data:** 1,563 annotated Bluesky and Twitter/X posts in English, Filipino, Cebuano and mixed language. After removing 6 duplicates, 1,557 posts were used and split 70/15/15 (seed 42) into about 1,089 training, 234 validation and 234 test posts.
- **Architecture:** `xlm-roberta-base` with the first-token vector and mean-pooled vector joined, followed by six output heads.
- **Training:** AdamW, encoder learning rate 2e-5, head learning rate 1e-4, batch size 16, up to 12 epochs, early stopping with patience 3, class-weighted losses and focal loss for disaster type.

### Classifier results (held-out test set)

| Head | Accuracy | Macro-F1 |
|---|---|---|
| Role | 0.9188 | 0.9008 |
| Urgency | 0.6410 | 0.6362 |
| Disaster type | 0.6923 | 0.5175 |
| Intent | 0.9145 | 0.9001 |

| Resource extraction metric | Value |
|---|---|
| Resources micro-F1 | 0.6947 |
| Resources macro-F1 | 0.6279 |
| Exact match | 0.4530 |
| Resource-count MAE | 0.4359 |
| Needs/offers micro-F1 | 0.6878 |

Role macro-F1 meets the 0.85 target. Disaster type is weaker because most training posts are labeled general disaster and few are earthquake or fire.

## Setup Instructions

> The thesis manuscript does not describe the repository layout or configuration files, so the paths, file names and environment variables below are **placeholders**. Adjust them to match the actual repo before publishing.

### Prerequisites

- Python 3.11
- Node.js 24.x and npm
- MongoDB 6.0 running locally
- A Bluesky account (used to sign in to the AT Protocol API)
- The calamanCy model `ljvmiranda921/tl_calamancy_md` (revision 0.2.0) and a local NLLB-200 checkpoint (`facebook/nllb-200-distilled-600M`)
- The trained classifier checkpoint (encoder, heads and tokenizer)
- A GPU is optional. The classifier uses CUDA automatically when available and runs on CPU otherwise.

### 1. Clone the repository

```bash
git clone https://github.com/bncnttt/AgapAI.git
cd AgapAI
```

### 2. Backend

```bash
cd backend                       # placeholder: use your actual backend folder
python -m venv .venv
.venv\Scripts\activate           # Windows (macOS/Linux: source .venv/bin/activate)
pip install -r requirements.txt  # placeholder
```

Create a `.env` file (placeholder names):

```env
BLUESKY_HANDLE=your-handle.bsky.social
BLUESKY_APP_PASSWORD=your-app-password
MONGODB_URI=mongodb://localhost:27017
```

Start the API:

```bash
uvicorn main:app --reload        # placeholder: use your actual app module
```

### 3. Frontend

```bash
cd frontend                      # placeholder: use your actual frontend folder
npm install
npm run dev
```

Open the local URL that Vite prints. The dashboard has no login of its own.

## Usage

1. Open the dashboard and choose a **date range**.
2. Click **Load Data**. The system returns stored records or fetches new posts for that period.
3. Review the results:
   - **Posts:** each post's role, urgency, needs and donations (filter by disaster type and role, or search by author handle)
   - **Users:** the accounts behind the posts
   - **Spatiotemporal Info:** resolved locations, confidence, ambiguity and event date and time
   - **Map View:** one pin per event on a map of Cebu
   - **SNA:** the incident queue of victims. Select a victim to see ranked volunteers, the paths connecting them and a social graph, then message a matched volunteer directly.

## Development Status

| Objective | Status |
|---|---|
| 1. Bluesky retrieval and Cebu filtering | Implemented, testing pending |
| 2. Multitask classification | Implemented, test results archived |
| 3. Resource extraction | Implemented, test results archived |
| 4. Spatiotemporal retrieval | Implemented, testing pending |
| 5. Event clustering | Implemented, testing pending |
| 6. SNA and volunteer matching | Implemented, no geographic distance yet |
| 7. Dashboard | Implemented, SUS and processing-time tests pending |
| 8. ISO/IEC 25010:2023 evaluation | Scheduled |

## Limitations

- **Single platform:** only Bluesky is covered, and its small Philippine user base limits how much Cebu disaster activity is visible.
- **On-demand updates:** by design, new posts appear only when a coordinator loads data again.
- **Model errors:** classification, geoparsing and matching can be wrong.
- **Public accounts only** and **Cebu only**.
- **No verification or delivery:** AgapAI suggests matches but does not verify reports or coordinate the delivery of help.
- **Uneven accuracy:** disaster-type (macro-F1 0.52) and urgency (0.64) are weaker than role and intent (about 0.90).
- **One intent per post**, so a post that both asks for and offers help is treated as only one.
- **Location evidence:** the gazetteer has no streets or landmarks at purok level, and a location taken from an author's bio or history shows where the author is, not necessarily where the incident is.
- **No distance in matching yet:** a socially close but distant volunteer can outrank a nearer one.
- **Bounded graph:** 3 hops, 500 relationships per account and 40 live expansions. A Bluesky follow only approximates real-world trust.

## Ethics and Data Privacy

AgapAI collects and stores public posts, public profile information and follow relationships of private individuals, including people who do not use the system. Only public data is collected. The study will document its compliance with the Data Privacy Act of 2012 (RA 10173) and Bluesky's developer terms, its data retention period, how post text and usernames are anonymized, and its ethics-review status before the manuscript is finalized. Do not publish collected posts or user data in this repository.

## Documentation

The full thesis manuscript (*AgapAI_MANUSCRIPT_REVISED.pdf*) covers the review of related works, software requirements and design, development process and testing plan. Add a link to it here if you host it.

## Recognition

Top 25, Philippine Startup Challenge.

## License

To be decided by the team. Add a `LICENSE` file before publishing.
