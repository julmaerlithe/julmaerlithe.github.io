# CERV (Community Emergency Reporting and Verification)

CERV is an app that utilizes a community-based alert system, empowering citizens to actively improve their communities by reporting issues, hazards, and emergencies in real time. Users can submit reports with photos, files, and GPS locations, which are then forwarded to the appropriate authorities for resolution. The system promotes transparency, enhances emergency response times, and strengthens collaboration between citizens and institutions, ultimately fostering safer and more resilient communities.

---

## Objectives

**Main Goal:**  
To provide a reliable and accessible platform that helps citizens report issues, emergencies, and hazards in real time.

**Specific Objectives:**
* Provide an alert system for the community, specifically in Cebu City.  
* Allow users to submit detailed reports with location and media, and notify users and authorities about real-time updates.  
* Facilitate faster responses from emergency departments while improving transparency and accountability in local issue resolution.

---

## Features

* **User Registration & Verification:** Create accounts, upload ID, selfie verification, and identity confirmation.  
* **Report Submission:** Submit reports with categories, descriptions, urgency, media (photos/videos), and GPS location.  
* **Report Tracking & Notifications:** Real-time updates on report status (pending, in-progress, dispatched, resolved).  
* **Admin Dashboard:** Authorities can review, categorize, assign actions, and generate monthly summaries.  
* **Hotspot & Map Visualization:** Display issues on a map with clustering of frequent concerns using Google Maps API.  
* **Emergency Response Alerts:** Forward urgent cases to nearby responders automatically.  
* **Media Repository:** Secure storage for photos, videos, and files.  
* **Basic Statistics & Reports:** Track monthly report counts and resolution status.

---

## System Design (High-Level)

**Modules / Architecture:**

* **User Module:** Registration, login, verification, profile management  
* **Report Module:** Report creation, media uploads, urgency tagging  
* **Notification Module:** Real-time alerts to users & responders  
* **Admin Module:** Dashboard for report review, status updates, and analytics  
* **Map & Hotspot Module:** Google Maps API integration with clustering  
* **Media Storage Module:** Secure cloud-based repository  
* **Statistics Module:** Weekly/monthly summaries, completed vs unresolved reports  

**Technologies / Tools:**

* Frontend: React.js / Flutter (mobile), HTML/CSS/JS (web)  
* Backend: Node.js / Express.js (or Java/Spring Boot)  
* Database: Firebase Firestore / MySQL / PostgreSQL  
* Map API: Google Maps API  
* Cloud Storage: Firebase Storage / AWS S3  
* Authentication: Firebase Auth / OAuth2  
* Notifications: Firebase Cloud Messaging / Twilio API

---

## Risks & Mitigation

**Possible Risks:**
* Fake or spam reports submitted by users  
* Server downtime during emergencies  
* Data privacy breaches (sensitive info exposed)  
* Low adoption rate by community and authorities  

**Mitigation Measures:**
* User verification with ID + selfie  
* Cloud hosting with load balancing and backup servers  
* Encryption of sensitive data and role-based access control  
* Community awareness campaigns and LGU staff training  

---

## Project Status

**Under Development**  
CERV is currently in its development phase. Core modules such as report submission, user authentication, and admin dashboards are being implemented. Future updates will include enhanced real-time notifications, analytics dashboards, and mobile integration.  

---

## Development Team

**Members:**
* Angelica D. Temones
* Jeramie Anne Q. Losorata
* Reiznick McCain G. Tenchavez

