# CivicAI — Real-World Upgrade Cheat Sheet

## 1. Project Overview

**Project Name:** CivicAI – AI-Powered Civic Intelligence & Grievance Management System

**Tagline:** Report. Verify. Resolve. Hold accountable.

### Problem Statement

Citizens report problems such as potholes, damaged roads, garbage accumulation, water leakage and pollution. However, many complaints are ignored, duplicated, incorrectly assigned or marked as resolved without actually fixing the problem.

### Our Solution

CivicAI is an AI-assisted civic accountability platform that allows citizens to report public infrastructure problems using photos, videos, voice or text.

AI analyzes complaints, estimates severity, recommends the responsible department and identifies duplicate reports.

The most important feature is **evidence-based resolution verification**. Complaints cannot simply be marked resolved without supporting evidence. Citizens can challenge an incorrect closure, helping prevent fake or incomplete repairs.

---

## 2. Main Features

### A. Citizen Complaint Reporting
- Citizen registration and login.
- Upload photos or videos of public problems.
- Submit complaints using text or voice.
- Automatically capture GPS location when permission is granted.
- Display complaint ID and submission status.
- Allow citizens to track their complaints.

### B. AI-Powered Complaint Analysis
Use Gemini Vision to analyze uploaded images.

AI identifies:
- Problem category.
- Type of damage.
- Visible defect size.
- Potential public safety hazards.
- Possible environmental risks.
- Severity score and explanation.
- Recommended department.
- Suggested action.

### C. Smart Duplicate Detection
When multiple citizens report the same issue:
- Detect potentially duplicate complaints.
- Compare location, category, image similarity and reporting time.
- Merge matching complaints into one master complaint.
- Preserve each citizen's original report and evidence.
- Increase priority when additional citizens confirm the issue.
- Notify citizens when their complaint is merged.

**Important:** Nearby complaints should not automatically be merged. The system must check whether they refer to the same actual problem.

### D. Evidence-Based Resolution
- Contractors or officers upload after-repair photos.
- Store original and after-repair evidence.
- Compare the before-and-after images.
- Check GPS and timestamps.
- Calculate verification confidence.
- Allow citizens to challenge suspicious closures.
- Send uncertain cases for human review.

### E. Admin Dashboard
- View all complaints.
- Filter by severity and department.
- Assign complaints to departments.
- Monitor SLA deadlines.
- Review duplicate complaints.
- Review evidence and citizen challenges.
- Reopen incorrectly resolved complaints.
- View complaint history and audit records.

### F. Live Complaint Map
Use Leaflet and OpenStreetMap to display:
- Complaint locations.
- Problem categories.
- Priority levels.
- Nearby duplicate reports.
- Open and resolved complaints.

### G. Offline Lite Reporting
When internet connectivity is unavailable:
- Allow citizens to submit a short text complaint.
- Capture location if available.
- Save the complaint locally for later submission.
- Mark it as `evidence_incomplete`.
- Request a photo after connectivity returns.

---

## 3. Technology Stack

| Component | Technology |
|---|---|
| Frontend | React.js + Tailwind CSS |
| Backend | Node.js + Express.js |
| Database | MongoDB |
| AI Analysis | Gemini API / Gemini Vision |
| Authentication | JWT + Firebase Auth (choose one approach) |
| Image Storage | Cloudinary |
| Maps | Leaflet + OpenStreetMap |
| Geolocation | Browser Geolocation API |
| Image Hashing | SHA-256 |
| Duplicate Detection | GPS + category + image similarity |
| Deployment | Vercel + Render or equivalent |

---

## 4. AI Severity Assessment and Smart Routing

This is one of CivicAI's main intelligence features.

### A. Severity Assessment

AI examines the complaint and identifies the factors that increase its severity.

Example: A large waterlogged pothole near a school.

The following factors are sorted from highest to lowest contribution:

```json
{
  "severity_assessment": {
    "base_score": 78,
    "weight_breakdown": [
      {
        "factor": "visible_defect_size",
        "contribution": "+25",
        "evidence": "Exposed aggregate approximately 1.5m wide"
      },
      {
        "factor": "proximity_to_school",
        "contribution": "+12",
        "evidence": "Nominatim identifies a government school within 80m"
      },
      {
        "factor": "standing_water_hazard",
        "contribution": "+8",
        "evidence": "Standing water obscures pothole depth for two-wheelers"
      }
    ],
    "final_score": 90,
    "priority_tier": "CRITICAL",
    "sla_hours": 24
  },
  "routing": {
    "primary_department": "Road Maintenance & Asphalt Division",
    "secondary_department": "Stormwater Drainage Cell",
    "routing_rationale": "Waterlogged pothole: primary road-surface defect with a secondary drainage contributor",
    "recommended_action": "Deploy a cold-mix patching crew and install reflective barricades immediately"
  }
}
```

**Note:** This is an illustrative AI output. The final score must be calculated by your backend's scoring rules, not blindly accepted from Gemini.

### B. Priority Levels

| Score | Priority | Suggested SLA |
|---|---|---|
| 0–20 | LOW | 7 days |
| 21–40 | MEDIUM | 5 days |
| 41–60 | HIGH | 3 days |
| 61–80 | VERY HIGH | 48 hours |
| 81–100 | CRITICAL | 24 hours |

These are proposed prototype values, not official government service standards.

### C. How the Score Works

1. Backend receives the AI's identified factors and evidence.
2. Backend validates each factor.
3. It applies predefined weights.
4. It calculates the final score.
5. It caps the score at 100.
6. It assigns the priority tier and suggested SLA.

Example calculation:

`78 + 25 + 12 + 8 = 123`

The final score is capped at 100.

For a more realistic scoring system, define a base score and weighted factors so that the final score does not exceed 100.

### D. Department Routing

Example:

| Problem | Primary Department | Secondary Department |
|---|---|---|
| Pothole | Road Maintenance | Stormwater Drainage |
| Garbage | Solid Waste Management | Public Health |
| Water Leakage | Water Supply Department | Road Maintenance |
| Streetlight Failure | Electrical Department | Public Safety |
| Air Pollution | Pollution Control Department | Public Health |
| Drainage Blockage | Drainage Department | Sanitation Department |

If Gemini returns low-confidence or invalid routing, use a predefined department mapping as a fallback.

**Important:** Department names and routing must be configurable for the target city. Do not claim official government integration without authorization.

---

## 5. Duplicate Complaint Management

### Problem

Suppose five citizens report the same pothole.

A normal complaint system may create five separate tickets. This creates unnecessary work and makes it difficult to understand the actual number of public problems.

### CivicAI Solution

1. Citizen A reports a pothole.
2. CivicAI creates master complaint `CIV-2026-8841`.
3. Citizen B reports the same pothole.
4. AI checks GPS, image similarity, category and time.
5. The system identifies a likely duplicate.
6. Citizen B's report is merged into the master complaint.
7. Original evidence is preserved.
8. The master complaint receives additional supporting evidence and a priority review.

### Citizen Notification

"Your report has been merged into master ticket CIV-2026-8841 as corroborating evidence. More citizens have reported this issue. The complaint priority has been updated. You will be notified when the repair is verified."

### Rules
- Never delete original reports.
- Preserve every citizen's contribution.
- Keep a merge history.
- Avoid merging different problems just because they are nearby.
- Use duplicate reports as supporting evidence, not proof that the issue is resolved.

---

## 6. Evidence-Frozen Resolution Verification

This is CivicAI's biggest differentiator.

### Problem

An officer or contractor may upload an unrelated or misleading after-repair photo and mark a complaint as resolved.

### Our Solution

CivicAI verifies the repair using:

1. Original complaint photo.
2. Original GPS location.
3. Original submission timestamp.
4. After-repair photo.
5. After-repair GPS location.
6. Image comparison.
7. Evidence hash.
8. AI verification result.
9. Citizen feedback.

### Evidence Chain

```json
{
  "evidence_chain": {
    "original_media_hash": "sha256-original-hash",
    "original_captured_at": "2026-11-15T09:00:00Z",
    "original_gps": {
      "lat": 12.9716,
      "lng": 77.5946
    },
    "after_media_hash": "sha256-after-hash",
    "after_gps": {
      "lat": 12.9717,
      "lng": 77.5947
    },
    "gps_offset_meters": 15,
    "frozen_at": "2026-11-15T12:00:00Z"
  }
}
```

The coordinates and timestamps above are illustrative.

### Important Verification Rules

- Require the after photo to be associated with the same complaint.
- Compare the original and after images.
- Check GPS distance and capture time.
- Flag suspicious location mismatches.
- Preserve the uploaded evidence and its SHA-256 hash.
- Do not assume that a matching GPS location proves a genuine repair.

A hash proves that the stored file has not changed since hashing. It does not prove that the original photo was genuine or captured at the claimed location.

---

## 7. Citizen Challenge Window

Sometimes AI may incorrectly decide that a repair is complete.

CivicAI gives citizens the opportunity to challenge a proposed closure.

### Confidence-Based Decisions

| Verification Confidence | Action |
|---|---|
| 0.85–1.00 | Eligible for automatic closure, subject to configured safeguards |
| 0.70–0.84 | Citizen challenge window |
| Below 0.70 | Human review |

These are prototype decision thresholds, not calibrated probabilities. Validate them against real test cases before using them for automatic decisions.

### Example

1. Contractor uploads an after-repair photo.
2. Gemini compares the original and after photos.
3. Verification confidence is 0.81.
4. CivicAI opens a 24-hour challenge window.
5. Citizens can challenge the proposed resolution.
6. If a valid challenge is submitted, the complaint is reopened.
7. The admin reviews the evidence and decides what happens next.

### Challenge Example

"The pothole is still visible. The uploaded after photo does not show the entire damaged area."

The system should not automatically penalize a contractor just because a citizen submits a challenge. Human review is required.

---

## 8. Complaint Closure State Machine

Every complaint follows a controlled lifecycle.

```text
IN_PROGRESS
     |
     | Officer uploads after-repair photo
     | GPS and evidence checks
     v
PENDING_VERIFICATION
     |
     | AI compares before and after evidence
     |
     |-------------------------------|
     |                               |
 Confidence >= 0.85           Confidence 0.70–0.84
     |                               |
     v                               v
AUTO_CLOSED                 CHALLENGE_WINDOW
                                     |
                              24-hour window
                                     |
                         |-----------|-----------|
                         |                       |
                    No challenge           Challenge received
                         |                       |
                         v                       v
                CITIZEN_CONFIRMED        CHALLENGED_REOPENED
                                                 |
                                                 v
                                             IN_PROGRESS

Confidence below 0.70
          |
          v
     HUMAN_REVIEW
          |
          v
  SUPER_ADMIN_DECISION
```

### Allowed Final States

- `AUTO_CLOSED`
- `CITIZEN_CONFIRMED`
- `CHALLENGED_REOPENED`
- `SUPER_ADMIN_OVERRIDE`

**Core rule:** A complaint must never become `RESOLVED` through an ordinary status update. It must pass an authorized verification or human-override path.

---

## 9. Database Structure

### A. Users Collection

```js
{
  _id,
  name,
  email,
  password_hash,
  role,
  created_at
}
```

Roles:
- CITIZEN
- OFFICER
- CONTRACTOR
- ADMIN
- SUPER_ADMIN

### B. MasterComplaints Collection

```js
{
  _id,
  complaint_id,
  citizen_id,
  category,
  description,
  location: {
    lat,
    lng,
    address
  },
  original_media_url,
  severity_assessment: {
    base_score,
    weight_breakdown,
    final_score,
    priority_tier,
    sla_hours
  },
  routing: {
    primary_department,
    secondary_department,
    routing_rationale,
    recommended_action
  },
  status,
  evidence_incomplete,
  evidence_chain: {
    original_media_hash,
    original_captured_at,
    original_gps: {
      lat,
      lng
    },
    after_media_hash,
    after_gps: {
      lat,
      lng
    },
    gps_offset_meters,
    frozen_at
  },
  verification: {
    is_same_location_match,
    is_issue_resolved,
    verification_confidence,
    verification_notes,
    challenge_window_ends_at,
    citizen_challenges: [
      {
        citizen_id,
        reason,
        created_at
      }
    ],
    final_state
  },
  merge_history: [
    {
      merged_complaint_id,
      merged_at,
      contributed_media_url
    }
  ],
  created_at,
  updated_at
}
```

### C. Challenges Collection

```js
{
  _id,
  complaint_id,
  citizen_id,
  reason_text,
  created_at,
  resolved_by,
  resolution
}
```

### D. Departments Collection

```js
{
  _id,
  department_name,
  categories,
  contact_details,
  active
}
```

### E. Updates Collection

```js
{
  _id,
  complaint_id,
  updated_by,
  old_status,
  new_status,
  message,
  created_at
}
```

### F. ContractorPenalties Collection (Optional)

```js
{
  _id,
  contractor_id,
  complaint_id,
  penalty_type,
  evidence,
  created_at,
  reviewed_by,
  review_status
}
```

Possible penalty types:
- `FAKE_CLOSURE_ATTEMPT`
- `GPS_MISMATCH`

Keep this as an investigation record in the prototype. Do not automatically impose penalties based on AI output.

---

## 10. System Architecture

```text
              CITIZEN / CONTRACTOR
                       |
                       v
                 REACT FRONTEND
                       |
                       v
                NODE + EXPRESS API
                       |
          +------------+------------+
          |            |            |
          v            v            v
      MONGODB       CLOUDINARY    AUTH SYSTEM
          |            |            |
          +------------+------------+
                       |
                       v
                  AI ENGINE
                       |
          +------------+------------+
          |            |            |
          v            v            v
      IMAGE AI     SEVERITY AI   ROUTING AI
          |            |            |
          +------------+------------+
                       |
                       v
              EVIDENCE ENGINE
                       |
          +------------+------------+
          |            |            |
          v            v            v
      GPS CHECK    IMAGE HASH    IMAGE VERIFY
                       |
                       v
              VERIFICATION ENGINE
                       |
          +------------+------------+
          |            |            |
          v            v            v
      AUTO CLOSE   CHALLENGE    HUMAN REVIEW
          |            |            |
          +------------+------------+
                       |
                       v
              ADMIN DASHBOARD
                       |
                       v
               CITIZEN TRACKING
```

---

## 11. Main Application Screens

### Screen 1: Landing Page
- CivicAI introduction.
- Report a problem button.
- How it works.
- Public complaint statistics.
- Login and registration.

### Screen 2: Citizen Dashboard
- My complaints.
- Complaint status.
- Priority level.
- SLA countdown.
- Notifications.

### Screen 3: Report a Problem
- Upload image/video.
- Enter description.
- Capture GPS.
- Select or confirm category.
- Submit complaint.

### Screen 4: AI Analysis
Display:
- Detected issue.
- Evidence explanation.
- Severity score.
- Priority tier.
- Recommended department.
- Recommended action.

### Screen 5: Admin Dashboard
- Total complaints.
- Open complaints.
- Critical complaints.
- Duplicate reports.
- Pending verification.
- Citizen challenges.

### Screen 6: Complaint Details
- Original complaint.
- Before-and-after photos.
- Location.
- AI analysis.
- Department assignment.
- Evidence history.
- Challenge option.
- Status updates.

### Screen 7: Complaint Map
- Interactive complaint pins.
- Category filters.
- Priority filters.
- Nearby complaint clusters.

---

## 12. Advanced Features

Build these only after the core workflow works.

1. Regional language support.
2. Voice-based complaint submission.
3. Duplicate complaint detection.
4. AI severity ranking.
5. Public complaint heatmap.
6. Offline Lite Report.
7. Before-and-after image verification.
8. Citizen challenge system.
9. Suspicious evidence review queue.
10. Complaint SLA monitoring.
11. Transparent complaint merge history.
12. Resolution statistics.

---

## 13. 18-Hour Hackathon Development Plan

### Phase 1: Core Setup
- Create React frontend.
- Set up Express backend.
- Connect MongoDB.
- Create citizen and admin login.
- Build complaint submission.

### Phase 2: AI Integration
- Connect Gemini Vision.
- Analyze uploaded complaint images.
- Detect the problem category.
- Generate severity evidence.
- Calculate score in the backend.
- Recommend department and action.

### Phase 3: Duplicate Detection
- Compare complaint locations.
- Add category and image similarity checks.
- Merge likely duplicates.
- Preserve original reports.
- Show merge notification.

### Phase 4: Evidence Verification
- Store original image hash.
- Allow officer to upload after photo.
- Check GPS distance.
- Compare before-and-after images.
- Implement verification confidence.
- Add citizen challenge window.

### Phase 5: Dashboard and Demo
- Build admin dashboard.
- Show complaint status.
- Display evidence history.
- Add complaint tracking.
- Test the complete flow.

### Must-Have Features

1. Complaint submission.
2. Real Gemini image analysis.
3. Explainable severity assessment.
4. Department recommendation.
5. Duplicate detection and merging.
6. Before-and-after evidence.
7. Citizen challenge window.
8. Admin dashboard and complaint tracking.

### Simplify or Cut if Time Is Limited

- Full video analysis.
- Full multilingual interface.
- Advanced heatmap.
- Complex OTP authentication.
- Automated contractor penalty system.
- Sophisticated analytics.

For a quick demo, use one regional-language voice example and a basic suspicious-evidence review queue.

---

## 14. Hackathon Demo Scenario

### Scenario 1: Duplicate Complaint

1. Citizen A reports a large pothole.
2. CivicAI analyzes the image.
3. The system assigns CRITICAL priority.
4. Citizen B reports the same pothole.
5. CivicAI detects a likely duplicate.
6. The second complaint is merged into the master ticket.
7. Both citizens can track the same master complaint.

### Scenario 2: Fake or Incomplete Repair

1. Admin assigns the complaint to a contractor.
2. Contractor uploads an after-repair photo.
3. CivicAI compares the original and after photos.
4. Verification confidence is 0.81.
5. A 24-hour challenge window opens.
6. Citizen reports that the pothole still exists.
7. The complaint is reopened.
8. The admin sees the challenge and evidence history.

**This is the strongest demo because it shows that CivicAI does more than detect problems: it helps ensure that reported problems are genuinely addressed.**

---

## 15. Performance and Impact Metrics

Display measurable results in the dashboard.

| Metric | What It Measures |
|---|---|
| Complaints analyzed | Total AI-analyzed reports |
| Analysis latency | Time taken for AI analysis |
| Classification accuracy | Correct problem categories |
| Duplicate detection | Correctly identified duplicate reports |
| Routing accuracy | Correct department recommendations |
| Verification accuracy | Correct repair verification decisions |
| Challenge rate | Share of proposed closures challenged |
| Reopened complaints | Complaints reopened after review |
| Resolution time | Time from submission to verified resolution |

Do not claim targets such as 100% routing accuracy or under 2.5-second analysis unless you have actually tested and measured them.

---

## 16. Real-World Safety and Trust Rules

- Never claim that CivicAI is officially connected to a government department unless permission exists.
- Label department routing as simulated in the prototype.
- Keep AI recommendations reviewable by humans.
- Do not treat Gemini confidence as a guaranteed probability.
- Do not merge complaints based on GPS alone.
- Do not treat GPS as proof of genuine image capture.
- Preserve original complaint evidence.
- Protect citizens' personal information and precise location.
- Restrict access to sensitive complaint and contractor records.
- Require human review before penalties or disputed final decisions.
- Use clear labels for simulated data and demo cases.

---

## 17. Project Differentiation

| Ordinary Complaint Website | CivicAI |
|---|---|
| Citizens submit complaints | Citizens submit photo, voice or text evidence |
| Manual categorization | AI-assisted classification |
| Basic priority selection | Explainable severity assessment |
| Duplicate tickets | Evidence-preserving duplicate merging |
| Simple resolved button | Evidence-based resolution verification |
| No citizen challenge | Citizen challenge window |
| Limited tracking | Full complaint audit history |
| Basic admin dashboard | AI-assisted civic accountability dashboard |

### Our Main Innovation

**Evidence-Frozen, Citizen-Challengeable Closure**

CivicAI does not allow a complaint to be silently closed just because someone changes its status. It preserves evidence, verifies the proposed repair and allows citizens to challenge an incorrect closure.

---

## 18. Final Hackathon Pitch

"CivicAI is an AI-assisted civic accountability platform designed to make public complaint resolution transparent and trustworthy.

Citizens can report potholes, garbage, water leakage and other infrastructure problems using images, voice or text.

Our AI analyzes the complaint, calculates its severity, recommends the responsible department and identifies duplicate reports.

But our biggest innovation is evidence-based resolution verification.

Every repair has before-and-after evidence, location checks and a verification process. If AI is uncertain, citizens can challenge the closure, and disputed cases are sent for human review.

CivicAI doesn't just help governments identify public problems. It helps ensure that problems are genuinely resolved."

### Final One-Line Identity

**CivicAI — AI-powered civic reporting with evidence-based accountability.**

---

## 19. API Quick Reference

The prototype API runs on the Express server. Protected routes use a bearer token returned by the login or registration endpoints.

### Public and Citizen Routes

| Method | Endpoint | Purpose | Access |
|---|---|---|---|
| `GET` | `/api/health` | Check API, database and AI configuration status | Public |
| `GET` | `/api/departments` | List prototype department mappings | Public |
| `GET` | `/api/complaints` | List complaints; supports `status` and `category` filters | Public or signed in |
| `POST` | `/api/auth/register` | Create a citizen account | Public |
| `POST` | `/api/auth/login` | Sign in and receive a token | Public |
| `GET` | `/api/auth/me` | Read the signed-in user's profile | Signed in |
| `POST` | `/api/analyze` | Analyze a description and optional image | Public |
| `POST` | `/api/complaints` | Submit a complaint and check for a likely duplicate | Optional sign in |
| `POST` | `/api/complaints/:id/challenges` | Challenge a proposed closure during its challenge window | Citizen |

### Staff Routes

| Method | Endpoint | Purpose | Access |
|---|---|---|---|
| `POST` | `/api/auth/bootstrap-admin` | Create the first super admin using the one-time bootstrap key | Initial setup only |
| `POST` | `/api/auth/provision` | Create staff accounts | Super admin |
| `POST` | `/api/complaints/:id/assign` | Assign a complaint to a department or staff member | Officer or admin |
| `POST` | `/api/complaints/:id/repair-evidence` | Submit after-repair photo and location for verification | Contractor, officer or admin |
| `POST` | `/api/complaints/:id/verification-decision` | Record an authorized final review decision | Admin or super admin |

### Authentication Header

For protected endpoints, send the token in the request header:

```http
Authorization: Bearer <token>
```

The complaint list limits citizen accounts to their own reports and reports they corroborated. Public responses hide photos, personal identifiers and precise coordinates. Staff access is role-checked by the API.

---

## 20. Local Demo Runbook

1. Install the project dependencies with `npm install`.
2. Start the frontend with `npm run dev`.
3. In a second terminal, copy `.env.example` to `.env` and start the API with `npm run server`.
4. Open the local Vite address shown in the terminal.
5. Register a citizen account and submit a sample issue.
6. For staff-only actions, configure the one-time admin bootstrap key, create the first super admin, then provision officer or contractor accounts.
7. Set `GEMINI_API_KEY` to enable Gemini analysis and repair-photo comparison. Without it, the prototype uses rule-based scoring and sends uncertain repair checks to human review.

Use clearly labeled demo complaints and simulated department names during presentations. Do not enter real citizen or sensitive personal information into the prototype.

---

## 21. Common Request Examples

### Analyze a Complaint

```http
POST /api/analyze
Content-Type: application/json
```

```json
{
  "category": "roads",
  "description": "A deep pothole is collecting water near the school crossing."
}
```

The API returns a severity assessment and department recommendation. With no Gemini key configured, the response identifies the rule-based fallback.

### Submit a Complaint

```http
POST /api/complaints
Content-Type: application/json
Authorization: Bearer <citizen-token>
```

```json
{
  "title": "Waterlogged pothole by school crossing",
  "category": "roads",
  "description": "A deep pothole is collecting water near the school crossing.",
  "location": {
    "lat": 19.076,
    "lng": 72.877,
    "address": "Demo location"
  },
  "media": []
}
```

For image evidence, include a `media` item with an image data URL in the format expected by the client. Keep uploads within the API's configured size limits. The response indicates whether this report created a new complaint or corroborated a likely duplicate.

### Filter the Complaint List

```http
GET /api/complaints?status=IN_PROGRESS&category=roads
```

Unauthenticated results are sanitized. A signed-in citizen sees their own reports and reports they corroborated; authorized staff can access the operations view.

---

## 22. Troubleshooting

| Symptom | Check |
|---|---|
| Frontend opens but API data does not load | Confirm the API is running with `npm run server` and check `GET /api/health`. |
| MongoDB connection is unavailable | Check `MONGODB_URI`; local demo mode uses in-memory data, which resets when the server stops. |
| AI output says fallback is active | Add a valid `GEMINI_API_KEY` to the server environment and restart the API. |
| Repair evidence goes to human review | Confirm AI comparison is configured and the repair submission includes a usable before image and after image; missing or uncertain GPS also prevents automatic approval. |
| Staff action returns unauthorized | Sign in with a staff account and send its bearer token; verify the account has the required role. |
| First super-admin setup is rejected | Set `ADMIN_BOOTSTRAP_KEY`, send the matching `x-admin-bootstrap-key` header, and confirm a super admin does not already exist. |
| Complaint disappears after server restart | In-memory mode is temporary; configure MongoDB for persistence. |
| Map tiles are blank | Check network access and the browser console; the map uses OpenStreetMap tiles. |

Never paste passwords, JWTs, bootstrap keys, API keys or private citizen evidence into public logs or issue reports.

---

## 23. Final Demo Readiness Checklist

- [ ] Frontend and API start successfully in separate terminals.
- [ ] `/api/health` responds and the expected storage and AI modes are understood.
- [ ] A citizen can register, sign in, submit a report and view its status.
- [ ] A second report can demonstrate duplicate detection without losing the original contribution.
- [ ] Staff accounts have the intended roles and can assign complaints.
- [ ] Repair evidence produces the expected verification path, including human review when evidence is uncertain.
- [ ] A citizen challenge can be demonstrated during a challenge window.
- [ ] The presentation labels prototype departments, sample records and simulated outcomes clearly.
- [ ] No real personal data or production credentials are included in the demo.

**End of cheatsheet.**
