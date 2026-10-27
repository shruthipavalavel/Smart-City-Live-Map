# Smart-City-Live-Map

### A Crowdsourced Smart Urban Resource Mapping System

Smart-City-Live-Map is an interactive web application that helps users explore urban resources and share real-time availability reports for EV charging stations, parking facilities, and public libraries across Bengaluru, India.

The application combines interactive mapping with crowdsourced reporting, trust-weighted status calculation, location-based reporting checks, and gamification. Users can report whether a resource is available or busy, earn experience points for their contributions, unlock badges, and compete on a contributor leaderboard.

The goal is to make urban resource information more useful through community participation while distinguishing between reported availability and resources whose status is unknown.

## Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Features](#features)
- [Technology Stack](#technology-stack)
- [System Architecture](#system-architecture)
- [Resource Categories](#resource-categories)
- [How It Works](#how-it-works)
- [Trust and Reputation System](#trust-and-reputation-system)
- [Gamification](#gamification)
- [Firebase Data Structure](#firebase-data-structure)
- [Security Considerations](#security-considerations)
- [Getting Started](#getting-started)
- [Firebase Configuration](#firebase-configuration)
- [Project Structure](#project-structure)
- [Limitations](#limitations)
- [Future Enhancements](#future-enhancements)
- [Attribution](#attribution)
- [License](#license)

## Overview

Urban resources such as parking facilities and EV charging stations may be easy to locate on a map, but their current availability can be difficult to determine.

Smart-City-Live-Map addresses this problem through a crowdsourced model. Users can submit availability reports for mapped resources, and the application combines recent reports to determine the displayed status.

The application also encourages participation through experience points (XP), achievement badges, contributor levels, and a leaderboard.

When recent reports are unavailable, a resource is displayed with an **Unknown** status instead of being incorrectly presented as available or busy.

## Objectives

- Provide an interactive map of urban resources in Bengaluru.
- Display EV charging stations, parking facilities, and public libraries.
- Enable authenticated users to submit availability reports.
- Update resource statuses as reports arrive in Firebase.
- Calculate trust estimates based on agreement between reports.
- Use trust-weighted voting to determine resource availability.
- Encourage participation through XP, levels, badges, and rankings.
- Apply location-based checks to discourage reports submitted far from a resource.

## Features

### 1. Interactive Resource Map

The application uses Leaflet and OpenStreetMap to display mapped resources across Bengaluru.

Each resource has a marker that displays its name, address, category, capacity, operator, current reported status, and recent reporting activity where available.

### 2. Multiple Resource Categories

The map supports three resource categories:

- EV charging stations
- Parking facilities
- Public libraries

Users can filter the map by category to explore the resources relevant to them.

### 3. Crowdsourced Availability Reporting

Authenticated users can submit reports indicating whether a resource is:

- **Available** — reported as available by a user.
- **Busy** — reported as busy by a user.
- **Unknown** — no qualifying recent reports are available.

Reports are stored in Firebase Realtime Database and include the resource identifier, user identifier, status, and submission timestamp.

### 4. Real-Time Report Synchronization

The application listens for changes to the Firebase reports collection using Realtime Database listeners.

When report data changes, the application refreshes resource markers and updates the displayed availability statistics.

### 5. Trust-Weighted Availability

Recent reports are combined using estimated reporter trust scores. Reports from users with higher estimated reliability receive greater weight when determining whether a resource is available or busy.

If a resource has no reports from the preceding 30 minutes, its status is displayed as Unknown.

### 6. Location-Based Reporting Checks

The application uses browser geolocation and a Haversine distance calculation to estimate the distance between the user and the selected resource.

The intended reporting radius is 100 metres. Reports from users detected beyond this distance are rejected.

If geolocation is unavailable or permission is denied, the current implementation allows the report to proceed. Therefore, the distance check is a deterrent rather than a strict guarantee of physical presence.

### 7. User Authentication

Firebase Authentication supports account creation, sign-in, and sign-out using email and password.

Users can create an account with a display name and access their contribution statistics.

### 8. Experience Points and Levels

Users earn XP for contributing availability reports. Additional XP is awarded for the first report at a resource and for every fifth report submitted.

The application includes five contributor levels:

1. Scout
2. Reporter
3. Verified
4. Guardian
5. Legend

### 9. Achievement Badges

The application includes achievement badges that reward participation and reporting milestones, including first reports, repeated contributions, reporting accuracy, and activity across resource categories.

### 10. Contributor Leaderboard

The leaderboard ranks users by their accumulated XP and displays contributor levels and rankings.

### 11. Personal Statistics

Users can view their XP, total reports, estimated reporting accuracy, current level, progress toward the next level, and earned badges.

### 12. Resource Status Statistics

The map interface displays counts of available, busy, and unknown resources, along with the number of reports submitted during the preceding 30 minutes.

## Technology Stack

| Component | Technology |
|---|---|
| Structure | HTML5 |
| Styling | CSS3 |
| Application logic | JavaScript (ES Modules) |
| Interactive mapping | Leaflet |
| Map tiles | OpenStreetMap |
| Authentication | Firebase Authentication |
| Database | Firebase Realtime Database |
| Location verification | Browser Geolocation API |
| Distance calculation | Haversine formula |
| Real-time synchronization | Firebase Realtime Database listeners |

Leaflet and Firebase are loaded through their respective CDN-hosted libraries in the current implementation.

## System Architecture

```text
                  ┌──────────────────────┐
                  │         User         │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │   Web Application    │
                  │                      │
                  │ HTML, CSS, JavaScript│
                  └──────────┬───────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
       ┌────────────┐ ┌────────────┐ ┌────────────┐
       │  Leaflet   │ │  Firebase  │ │ Geolocation│
       │    Map     │ │            │ │    API     │
       └─────┬──────┘ ├────────────┤ └────────────┘
             │        │ Auth       │
             ▼        │ Realtime DB│
       ┌────────────┐ └─────┬──────┘
       │ OpenStreet │       │
       │    Map     │       ▼
       └────────────┘ ┌────────────┐
                      │ Reports and│
                      │ User Stats │
                      └─────┬──────┘
                            │
                            ▼
                   ┌─────────────────┐
                   │ Trust-Weighted  │
                   │ Status & XP     │
                   └─────────────────┘
```

## Resource Categories

The current implementation contains 19 predefined resource entries.

| Category | Number of locations |
|---|---:|
| EV charging stations | 10 |
| Parking facilities | 6 |
| Public libraries | 3 |
| **Total** | **19** |

These are predefined application records. Their presence on the map does not independently verify the operational status or actual capacity of each location.

## How It Works

### Step 1: User Authentication

Users register or sign in through Firebase Authentication. The application loads their saved contribution statistics from Firebase.

### Step 2: Resource Display

The application initializes a Leaflet map centered on Bengaluru and places markers for the predefined resources.

OpenStreetMap supplies the map tiles, while the application maintains the resource details and coordinates.

### Step 3: Availability Reporting

Users select a resource marker and choose Available or Busy.

The application checks the user's location when browser geolocation is available and compares the calculated distance with the 100-metre reporting radius.

Accepted reports are written to Firebase Realtime Database.

### Step 4: Real-Time Updates

A Firebase listener retrieves report data and updates the local report collection. The application refreshes markers and recalculates the resource status and summary statistics.

### Step 5: Trust-Weighted Status Calculation

The application compares reports for the same resource submitted within a 15-minute window to estimate reporter trust. Recent reports from the preceding 30 minutes are then combined using these trust estimates.

If no qualifying recent reports exist, the resource is marked Unknown.

### Step 6: Rewards and Statistics

Submitting a report increases the user's report count and awards XP. First-report and reporting-streak bonuses are also applied where applicable.

The application updates the user's level, badges, statistics, and leaderboard information.

## Trust and Reputation System

Smart-City-Live-Map uses a trust-weighted voting mechanism to estimate resource availability.

### Trust Estimation

For each resource, reports from the last 30 minutes are combined using the reporters' estimated trust scores.

The weighted vote for availability is calculated as:

$$
V_{\text{available}} = \sum_{r \in A} T_{u(r)}
$$

The weighted vote for busy status is calculated as:

$$
V_{\text{busy}} = \sum_{r \in B} T_{u(r)}
$$

Here:

- \(A\) represents the set of recent reports indicating that the resource is available.
- \(B\) represents the set of recent reports indicating that the resource is busy.
- \(T_{u(r)}\) represents the estimated trust score of the user who submitted report \(r\).

The status with the higher weighted vote is selected. If no qualifying recent reports exist, the resource is marked as **Unknown**.



### Trust-Weighted Voting

The trust-weighted votes are calculated as follows.

**Available status:**

$$
V_{\text{available}} = \sum_{r \in A} T_{u(r)}
$$

**Busy status:**

$$
V_{\text{busy}} = \sum_{r \in B} T_{u(r)}
$$

Where:

- \(V_{\text{available}}\) is the total trust weight of reports indicating availability.
- \(V_{\text{busy}}\) is the total trust weight of reports indicating that the resource is busy.
- \(A\) is the set of recent reports with status `available`.
- \(B\) is the set of recent reports with status `busy`.
- \(T_{u(r)}\) is the trust score of the user who submitted report \(r\).

The application selects the status with the higher total trust weight. If no qualifying reports exist within the last 30 minutes, the resource is marked as **Unknown**.


**Limitation:** Agreement between users is only a proxy for accuracy. The current implementation does not independently verify whether a report reflects the resource's actual condition. Trust scores are calculated from the available reports rather than maintained as a separately validated ground-truth reputation system.

## Gamification

The application encourages community participation through an XP and achievement system.

| Activity | XP reward |
|---|---:|
| Submit a report | +10 XP |
| First report at a resource | +5 bonus XP |
| Every fifth report | +20 bonus XP |
| Report confirmed by the crowd | +15 XP shown in the interface |

The first three rewards are implemented in the report-submission logic. The crowd-confirmation reward is displayed in the interface, but it should not be considered implemented unless a corresponding confirmation mechanism is added to the application logic.

### Contributor Levels

| Level | Required XP |
|---|---:|
| Scout | 0 |
| Reporter | 50 |
| Verified | 150 |
| Guardian | 350 |
| Legend | 700 |

### Achievement Badges

The application defines badges for:

- First Report
- On a Roll
- Committed
- Sharp Eye
- Trusted
- EV Champion
- Parking Pro
- Library Scout
- All-Rounder

Badges are awarded according to the contribution and progress conditions defined in the application.

## Firebase Data Structure

The application uses Firebase Realtime Database for reports and user statistics.

### Reports

Reports are stored under the `reports` path.

```json
{
  "reports": {
    "generated_report_id": {
      "resourceId": "ev_001",
      "userId": "firebase_user_uid",
      "status": "available",
      "createdAt": 1790000000000
    }
  }
}
```

The report identifier is generated by Firebase when a new report is pushed.

### Users

User statistics are stored under `users/{uid}`.

```json
{
  "users": {
    "firebase_user_uid": {
      "xp": 10,
      "total": 1,
      "correct": 0,
      "evRep": 1,
      "parkRep": 0,
      "libRep": 0,
      "earnedBadges": ["first"],
      "displayName": "Example User",
      "lastSeen": 1790000000000
    }
  }
}
```

The `correct` field is used in the displayed accuracy calculation, but the current reporting flow does not update it when a report is confirmed. Consequently, the displayed accuracy should not be interpreted as a fully validated accuracy metric until that logic is implemented.

### Resource Data

Resource details are currently defined in a JavaScript array in the application code. Each record contains a resource ID, name, category, coordinates, capacity, operator, and address.

The current implementation does not require a separate Firebase `resources` collection to display these predefined markers.

## Security Considerations

The application includes authentication and client-side location checks, but these do not by themselves guarantee that all reports are trustworthy.

Recommended security measures include:

- Configure Firebase Realtime Database Security Rules to restrict reads and writes.
- Require authenticated users to submit reports.
- Validate report status values, resource identifiers, and timestamps.
- Apply server-side rate limiting to reduce spam and repeated submissions.
- Prevent users from directly changing XP, trust scores, or other users' statistics.
- Move sensitive reputation and reward calculations to trusted backend logic where appropriate.
- Treat browser geolocation as an estimate because location spoofing is possible.
- Avoid exposing service-account credentials or other server-only secrets in client-side code.
- Review what user information is stored and restrict access to it.

The current implementation performs important reporting, trust, and reward calculations in frontend JavaScript. These checks can be manipulated by a determined user, so backend enforcement would be needed before treating the application as production-secure.

## Getting Started

### Prerequisites

- A modern web browser
- A Firebase project
- Internet connectivity for the CDN libraries and map tiles
- A local development server

### 1. Clone the Repository

Replace the placeholder with the actual repository URL.

```bash
git clone <your-repository-url>
cd Smart-City-Live-Map
```

### 2. Open the Application

If the application is stored in `index.html`, run it through a local development server. For example, if Python is installed:

```bash
python -m http.server 8000
```

Open the address below in your browser:

```text
http://localhost:8000
```

A local server is preferable to opening the HTML file directly because the application uses JavaScript modules and Firebase services.

### 3. Configure Firebase

Create a Firebase project, enable Email/Password authentication, and create a Realtime Database.

Copy the Firebase web configuration into the `firebaseConfig` object in the JavaScript module.

## Firebase Configuration

The application expects the following configuration fields:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "YOUR_DATABASE_URL",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

Use the actual values from your Firebase project.

Configure appropriate Realtime Database Security Rules before allowing users to submit reports. The Firebase web configuration identifies the project; it does not replace access control.

Never put Firebase Admin SDK credentials or service-account private keys in the client-side application.

## Project Structure

The current implementation can be organized as follows:

```text
Smart-City-Live-Map/
│
├── index.html
└── README.md
```

The HTML file contains the application's markup, styling, map initialization, Firebase integration, authentication flow, reporting logic, trust calculation, XP system, badges, and leaderboard.

If the project is split into separate files in the future, update this section to reflect the actual repository structure.

## Limitations

- The application contains 19 predefined resource entries, rather than automatically importing all urban resources.
- Resource coordinates and details are not independently verified by the application.
- OpenStreetMap provides map tiles, not live resource availability.
- Availability depends on user-submitted reports.
- Reports may be inaccurate, outdated, or manipulated.
- The trust calculation relies on agreement between reports rather than verified ground truth.
- The location check can be bypassed when geolocation is unavailable or permission is denied.
- XP, trust estimates, and other client-side calculations require backend enforcement to resist manipulation.
- The current implementation does not independently confirm whether a resource is actually available.

## Future Enhancements

- [ ] Enforce location verification through trusted backend logic.
- [ ] Move XP, trust, and reward calculations to secure backend logic.
- [ ] Add server-side report rate limiting and duplicate-report prevention.
- [ ] Implement a reliable mechanism for confirming report accuracy.
- [ ] Improve the calculation of contributor accuracy.
- [ ] Integrate verified resource and availability data sources where available.
- [ ] Add resource search and navigation.
- [ ] Improve mobile responsiveness and accessibility.
- [ ] Expand the resource dataset beyond the initial 19 locations.
- [ ] Add historical availability summaries based on collected reports.
- [ ] Add automated tests for reporting, trust calculation, and authentication.

## Attribution

Map tiles are provided by OpenStreetMap contributors.

© OpenStreetMap contributors.

For details, see [OpenStreetMap Copyright and License](https://www.openstreetmap.org/copyright).

The application also uses Leaflet and Firebase. Follow the applicable terms and usage policies for these services.

## License

No license has been specified yet. Add a `LICENSE` file and update this section when a license is selected.

## Author

**Shruthi Pavalavel**

Smart-City-Live-Map explores crowdsourced urban resource mapping, trust-weighted reporting, location-based checks, and gamified community participation.
