# Exam Hub

> One link. Every exam resource. Organised by the topic your child is actually studying.

📹 **Watch the demo video:** https://youtu.be/_pcaVPZMsR8

> **Note on privacy:** The live app link, HTML source, and all linked study files are not included in this repository. The project contains real school documents and student-relevant content that is sensitive in nature — sharing it publicly would not be appropriate. This README exists purely to document the product, the problem it solved,  the technical approach to encourage others to solve similar problems.

---

## 📖 "The What" — What is the product?

Exam Hub is a single-page web app built for parents of students ahead of their Term exams. It aggregates close to 150 study resources — worksheets, PDFs, answer keys, revision sheets, videos — from multiple school sources and organises them by exam topic, mirroring the exact syllabus structure.

Click on any topic and you instantly see every document the teacher shared for that subject. Search across all five subjects at once. Everything is a real school link — nothing is copied or stored locally.

It works on any device. Desktop for parents reviewing at home, mobile for parents on the go.

---

## 🤔 "The Why" — The Problem It Solves

With five exams in a single week, study resources were scattered across at least three different places:

1. **Class Drive** — a folder hierarchy with hundreds of files across all subjects and topics
2. **Daily class diary** — an Excel sheet per month, a tab per week, individual days linking to materials used in class
3. **School app newsfeed** — a social-feed-style interface where teachers occasionally share documents outside the Drive

None of these surfaces answers the question a parent actually asks: *"For this specific topic in tomorrow's exam, which documents do I need?"*

That question was being asked constantly in the parents' WhatsApp group. No one had a clean answer.

---

## ❤️ "The Impact" — What Parents Are Saying

The hub was shared with the Grade 4 parents' WhatsApp group and saw strong organic adoption within days:

- **~160 unique users** across devices
- **~500 sessions** in 1 week

Here's a sample of what parents shared in the group:

> "I usually keep track of all the documents sent by the school, but I still seem to have missed a few from your list. Everything has been put together so well — thank you for your effort. It's definitely very helpful!!"

> "This is awesome!!! Realizing that haven't even looked at half of the worksheets!!"

> "At least at 1 glance we can see what we still have to accomplish in each subject 🫣"

> "Thanks for taking extra time to work on this, really appreciate"

> "Thank you for taking out the time !! It's commendable !"

> "Thanks a ton for your time and efforts."

> "I hv missed a few of it too"

> "Wow super helpful thanks a lot !"

> "Super work, Pradeep. Tx so much!"

> "Offer freelancing to school.. at least some help to parents 🙈"

---

## 🔧 "The Hard Parts" — Challenges & Learnings

### Extracting the Drive folder hierarchy

The class Drive folder had a deeply nested structure. Google Drive's web interface is JS-rendered, so standard scraping doesn't work. The solution was a Google Apps Script that enumerates folder contents programmatically, flattens the hierarchy, and outputs a structured document. That output was fed into the LLM to map each file to its corresponding syllabus topic. Close to 150 files were organised this way.

### The school app newsfeed — the blind spot

Not all resources live in Drive. Some teachers share documents through a newsfeed-style UI in the school app — a surface with no export, no API, and no easy way to scan programmatically. For this version, the last two months of posts were manually reviewed to catch anything relevant. The real fix would be a scraper that crawls the last six months of posts (covering the full syllabus period) and surfaces shared documents automatically. That's the next step.

### Analytics on a static site

Since the hub is a static HTML file on Google Cloud Storage, there's no server to log requests. A lightweight Cloud Run service backed by Firestore was built specifically to handle this — a 1×1 pixel image request on every page load logs the session, and a protected stats dashboard shows unique users, sessions, and visit timelines.

### CDN caching

GCS defaults to `Cache-Control: public, max-age=3600`. Pushing an update without the `--cache-control` flag meant parents were seeing the stale version for up to an hour — even in incognito. Every deploy now explicitly passes `--cache-control="no-cache, no-store, must-revalidate"`.

---

## ⚙️ "The How" — Tech Architecture

| Layer | Technology | Why |
|-------|-----------|-----|
| Hosting | Google Cloud Storage (static website) | Zero infra — single HTML file, no server needed |
| Analytics backend | Google Cloud Run (Python / Flask) | Lightweight, scales to zero, same GCP project |
| Analytics storage | Google Firestore | Handles concurrent writes cleanly, free tier sufficient |
| Session tracking | `localStorage` (user ID) + `sessionStorage` (session ID) | Persistent user identity across visits, session reset on browser close |
| Drive extraction | Google Apps Script | Only way to enumerate JS-rendered Drive folder contents |
| Data sources | Google Drive, Google Sheets (class diary) | All original school links — no content copied |
| Deployment | `gcloud storage cp` + `gcloud run deploy` | Full deploy in under 2 minutes |

The entire front-end is a single self-contained HTML file — no external CSS frameworks, no JS libraries, no build pipeline. All styles and scripts are embedded inline.

---

*Built by Pradeep Elavarasan · Co-created with Claude*
