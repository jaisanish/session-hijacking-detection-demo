# 🎓 Teacher / Evaluator Demo Walkthrough & Presentation Guide

> **Project:** Behavioral Fingerprint-Based Detection of Session Hijacking  
> **Tech Stack:** Python 3 (Flask), Vanilla JavaScript, HTML5/CSS3 (In-Memory Session Store)  
> **Evaluation Focus:** Web Application Security, Session Management, Heuristic Anomaly Detection  

---

## 📌 Executive Summary for the Evaluator

In traditional web authentication, session tokens (stored in cookies or local storage) are **bearer tokens**: *whoever possesses the token is authenticated as that user*. If an adversary steals this token via Cross-Site Scripting (XSS), insecure Wi-Fi sniffing, or physical machine access, standard servers cannot distinguish the legitimate user from the attacker.

**Our Solution:**  
This project binds every active session token to a **client behavioral fingerprint** (User-Agent, display resolution, and timezone). Whenever a request with a valid token reaches the server, the server compares the incoming client's device fingerprint with the fingerprint recorded at initial login. If more than **50% of the fingerprint fields mismatch**, the system flags an impersonation attempt, logs the anomaly in real time, and **immediately revokes the session on the server**.

---

## 🖥️ UI & Feature Walkthrough (Every Page & Button)

### 1. The Login Page (`/login`)

| Element / Button | Technical Function | Presentation Note |
|---|---|---|
| **Header Banner** | Displays title: *"Session Hijacking Demo — Behavioral Fingerprint-Based Detection"*. | Sets the context for the evaluator immediately. |
| **Username Field** | Pre-filled with demo user `student`. | Standard credential input. |
| **Password Field** | Pre-filled with demo password `student123`. | Password input. |
| **Hidden Input Fields** (`#fp_user_agent`, `#fp_screen_width`, `#fp_screen_height`, `#fp_timezone`) | Injected silently into the POST body via client-side JavaScript when the form submits. | **Key viva point:** Point out that this requires **zero user friction**—no captchas or extra OTPs. |
| **"Log In" Button** | Executes the form submission, triggering JS fingerprint capture and POSTing credentials + fingerprint to `/login`. | Explain: On click, client attributes are collected and securely posted. |

---

### 2. The Protected Dashboard (`/dashboard`)

Once logged in, the user is redirected to the main dashboard. It contains four core components:

#### A. Header & Logout
- **"Protected Dashboard" Title**: Confirms active authenticated state.
- **"Logout" Button**: Sends `POST /logout`, revokes the session token from the server's `SESSIONS` dictionary, clears the browser cookie, and logs a `LOGOUT` event.

#### B. Session Information Card
- **Logged in as:** Displays the authenticated identity (`student`).
- **Session Status Badge:** Green `Active` badge indicating valid status on the server.
- **Fingerprint Status Badge:** Shows `Matched` when the current request matches the stored fingerprint.
- **Session Created:** Exact UTC/local timestamp when the token was issued.
- **Fingerprint Hash:** A 16-character SHA-256 digest of the combined fingerprint attributes (`hashlib.sha256(raw).hexdigest()[:16]`). Demonstrates cryptographic integrity checking.

#### C. Browser Fingerprint Card (Captured at Login)
- **User Agent:** Full browser engine, OS version, and platform string (`navigator.userAgent`).
- **Screen Resolution:** Hardware width × height (e.g., `1920 x 1080`).
- **Timezone:** Resolved IANA timezone string (e.g., `Asia/Kolkata`).

#### D. Demo Controls Card (The Live Testing Center)
- **Current Session Token Box:** Monospace display showing the 43-character cryptographically secure token (`secrets.token_urlsafe(32)`).
- **"Copy Token" Button:** Copies the token to the clipboard and shows a green confirmation message (`"Token copied to clipboard"`). Simulates an attacker extracting a stolen session cookie.
- **Test Page URL Display:** Shows `http://127.0.0.1:5050/demo/test-session`.
- **"Open Test Page (this browser)" Button:** Opens the test page in the current browser to allow demonstrating the **baseline non-hijack scenario**.

#### E. Server Event Log Card (Real-Time Security Feed)
- **Auto-Refreshing Table:** Polls `GET /api/events` every 5 seconds without full page reload.
- **Event Types & Color-Coded Badges:**
  - `LOGIN` (Green) — Successful user login and initial fingerprint registration.
  - `LOGIN_FAIL` (Red) — Invalid credentials rejected.
  - `FP_MATCH` (Blue) — Fingerprint verified and authorized.
  - `SUSPICIOUS` (Yellow) — Anomaly detected during dashboard access.
  - `HIJACK_BLOCKED` (Red) — Hijacking attempt detected; session instantly revoked.
  - `LOGOUT` (Gray) — Clean user session termination.

---

### 3. The Attacker Simulation Page (`/demo/test-session`)

This page acts as the **adversary's workstation**. It allows anyone to present a session token without having the original victim's cookie file.

| Element / Button | Technical Function | Presentation Note |
|---|---|---|
| **Session Token Textarea** | Multi-line input where the stolen token is pasted. | Simulates an attacker injecting a stolen cookie into their own request. |
| **Hidden Fingerprint Fields** | JavaScript on this page captures the *attacker's* browser attributes (`navigator.userAgent`, `screen.width`, `screen.height`, `Intl.DateTimeFormat`). | These values represent the attacker's hardware environment. |
| **"Test Session" Button** | Submits the stolen token + attacker fingerprint to `POST /demo/test-session`. | The moment of detection. |
| **Side-by-Side Comparison Table** | Rendered upon submission: compares **Original (Login Browser)** vs. **Incoming (This Browser)** across all 4 attributes with a `Yes`/`No` match status. | Demonstrates clear visual proof of discrepancy to the evaluator. |
| **Mismatch Ratio Banner** | Displays percentage of mismatched fields (e.g., `75% of fields differ. Threshold: >50% triggers detection`). | Proves the threshold algorithm is functioning mathematically. |
| **"Go to Login" Button** | Navigates back to the root login page after revocation. | Reset button. |

---

### 4. The Access Blocked Page (`/blocked`)
- Displayed if the victim or attacker attempts to access protected routes using a flagged/revoked token.
- Explains that the session has been terminated due to suspicious behavioral variance.

---

## 🎬 5-Minute Presentation & Demo Script

Follow this structured script when demonstrating the project to your teacher or examiner.

### Minute 1: The Problem Introduction
> *"Good morning/afternoon, Sir/Ma'am. Today, we are demonstrating **Behavioral Fingerprint-Based Detection of Session Hijacking**.*  
> *In traditional web systems, cookies are bearer tokens. If an attacker intercepts your session token via public Wi-Fi sniffing or XSS, they can paste that cookie into their browser and gain complete access to your account.*  
> *Our project solves this vulnerability without adding friction like repetitive 2FA codes. We bind the session token to the user's browser fingerprint at login and dynamically verify it on every subsequent request."*

### Minute 2: Victim Login & Fingerprint Registration
> 1. Open Chrome at `http://127.0.0.1:5050`.
> 2. Show the pre-filled credentials (`student` / `student123`).
> 3. Click **"Log In"**.
> 4. Point to the **Dashboard**:
>    - *"Notice how the server captured my User-Agent, my resolution (e.g., 1920x1080), and my timezone (Asia/Kolkata). It computed a unique SHA-256 fingerprint hash."*
>    - *"Notice the Event Log at the bottom showing a green `LOGIN` event."*

### Minute 3: Simulating Session Theft (The Attack)
> 1. Click the **"Copy Token"** button on the dashboard.
>    - *"I am now acting as an attacker who has extracted this session token."*
> 2. Open a completely different browser (e.g., Firefox, Edge, or a mobile browser on the same Wi-Fi).
> 3. Navigate to `http://127.0.0.1:5050/demo/test-session`.
> 4. Paste the stolen token into the box.
> 5. Click **"Test Session"**.

### Minute 4: The Detection & Instant Revocation
> 1. Show the screen in Browser B:
>    - Point to the **red alert:** `🚨 Detection Result: Hijacking Detected`.
>    - Point to the **comparison table:** *"Look at the table: Browser B's User-Agent is Firefox, while the original was Chrome. The mismatch ratio is 75%, which exceeds our 50% threshold. The server has instantly invalidated the token."*
> 2. Bring **Browser A (Chrome)** back to view:
>    - Watch the **Server Event Log** update live: a red `HIJACK_BLOCKED` event appears!
>    - Refresh Browser A: the victim is immediately kicked out to `/login` because the compromised session was revoked globally on the server.

### Minute 5: False-Positive Baseline Test & Conclusion
> 1. Log in again in Chrome to generate a fresh token.
> 2. Open `http://127.0.0.1:5050/demo/test-session` in another tab in the **same browser**.
> 3. Paste the token and click **"Test Session"**.
> 4. Show the result:
>    - Green/Yellow message: *"Fingerprint matched. This appears to be the same browser — no hijacking detected."*
> 5. Conclusion:
>    - *"This proves our heuristic avoids false alarms when the legitimate user switches tabs, while strictly blocking cross-device or cross-browser impersonation."*

---

## 💡 Anticipated Teacher / Viva Q&A

### Q1: Why not just bind the session to the client's IP address?
**Answer:**  
*"Binding solely to IP addresses introduces severe usability issues:*
- *Legitimate users on mobile devices frequently switch IP addresses when moving between Wi-Fi and 4G/5G cellular towers, causing abrupt logouts.*
- *Multiple users behind the same NAT gateway, university proxy, or corporate VPN share the exact same public IP address. An attacker on the same campus network would have the same IP as the victim!*  
*Browser fingerprinting inspects device-level characteristics independent of network routing."*

### Q2: What happens if a user resizes their browser window? Does it cause a false positive?
**Answer:**  
*"In our implementation, we use `screen.width` and `screen.height` (the physical display monitor resolution), not `window.innerWidth` / `window.innerHeight`. Window resizing does not alter screen dimensions.*  
*Furthermore, our algorithm uses a **>50% mismatch threshold** across 4 attributes. Even if one attribute fluctuates, 1 mismatch out of 4 is only 25%, safely below the 50% threshold."*

### Q3: How is the fingerprint stored on the server?
**Answer:**  
*"The server maintains an in-memory dictionary `SESSIONS[token]` storing:*
1. *The raw attribute dictionary (`user_agent`, `screen_width`, `screen_height`, `timezone`).*
2. *A 16-character SHA-256 hash digest of the attributes for rapid integrity checks.*
3. *A state enum (`active`, `suspicious`, `revoked`).*  
*In an enterprise production deployment, this in-memory dictionary would be replaced with a distributed **Redis** or **Memcached** store with TTL expirations."*

### Q4: Can a skilled attacker spoof the client-side JavaScript fingerprint?
**Answer:**  
*"If an attacker only steals the cookie over the wire (network packet capture or MITM), they don't know the victim's exact screen dimensions or timezone.*  
*However, in an advanced scenario where an attacker has full client-side XSS, they could potentially inspect and spoof these values. In production, we counter this with:*
- *Server-side TLS/JA3/JA4 fingerprinting (inspecting cipher suites in the TCP/TLS handshake which JS cannot alter).*
- *Canvas & WebGL rendering benchmarks.*
- *AudioContext hardware latency measurements.*
- *HTTP-only cookies to prevent cookie theft via XSS in the first place."*

### Q5: What is the computational overhead of this defense?
**Answer:**  
*"Virtually zero. The client-side collection uses standard DOM queries (`navigator` and `screen` objects) which execute in sub-millisecond time (<1ms). The server-side comparison is a simple dictionary key lookup and four string equality checks ($O(1)$ constant time)."*
