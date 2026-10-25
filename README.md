# 🔐 Behavioral Fingerprint-Based Detection of Session Hijacking

> **A real-time, interactive cybersecurity demonstration built with Python (Flask) and JavaScript.**  
> Runs 100% locally with zero external dependencies, no database required, and in-memory session tracking.

---

## 📌 Problem & Solution Overview

### The Vulnerability: Session Hijacking (Bearer Token Abuse)
In traditional web applications, cookies and session tokens are **bearer tokens**: *anyone possessing the token is granted access as that user*. If an attacker intercepts or steals this token via:
- Cross-Site Scripting (XSS)
- Public or untrusted Wi-Fi packet sniffing (MitM)
- Browser infostealer malware or unauthorized local access

...the standard web server blindly accepts the stolen token and grants full access to the attacker.

### Our Solution: Behavioral Browser Fingerprinting
This application binds every generated session token to the user's **client browser fingerprint** (User-Agent, display resolution, and timezone) at login. 
- On every subsequent request or test, the server compares the client's current hardware environment with the baseline recorded at login.
- If **more than 50% of the fingerprint attributes mismatch**, the system detects session hijacking, logs the incident to a real-time event feed, and **instantly revokes the session across the server**.

---

## ⚡ Quick Start Guide (Run Locally)

### Prerequisites
- **Python 3.8+** installed (`python --version`)
- A modern web browser (Chrome, Firefox, Edge, Safari)

---

### Step 1: Clone the Repository
```bash
git clone https://github.com/jaisanish/session-hijacking-detection-demo.git
cd session-hijacking-detection-demo
```

---

### Step 2: Set Up Virtual Environment & Dependencies

#### 🪟 On Windows (PowerShell)
```powershell
# 1. Create virtual environment
python -m venv venv

# 2. Activate virtual environment
.\venv\Scripts\Activate.ps1

# 3. Install dependencies
pip install -r requirements.txt
```

> **Note for PowerShell Users:** If you see an execution policy error on `Activate.ps1`, either run:
> ```powershell
> Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
> .\venv\Scripts\Activate.ps1
> ```
> *Or run directly without activating:*
> ```powershell
> .\venv\Scripts\python.exe -m pip install -r requirements.txt
> .\venv\Scripts\python.exe app.py
> ```

#### 🪟 On Windows (Command Prompt / CMD)
```cmd
python -m venv venv
venv\Scripts\activate.bat
pip install -r requirements.txt
```

#### 🍎 On macOS / 🐧 Linux
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

### Step 3: Run the Application

```bash
python app.py
```

Open your browser and navigate to:  
👉 **[http://127.0.0.1:5050](http://127.0.0.1:5050)**

### 🔑 Demo Credentials
| Field | Default Value |
|---|---|
| **Username** | `student` |
| **Password** | `student123` |

---

## 🧪 Step-by-Step Live Attack Demonstration

Demonstrate the attack and defense mechanism in 3 easy steps:

```
+------------------------------------+        +------------------------------------+
|  Browser A (e.g. Chrome)           |        |  Browser B (e.g. Firefox)          |
|  VICTIM USER                       |        |  ATTACKER                          |
+------------------------------------+        +------------------------------------+
| 1. Log in at /login                |        |                                    |
| 2. Dashboard registers Fingerprint |        |                                    |
| 3. Click [Copy Token] -------------> (Steals) -> 4. Open /demo/test-session      |
|                                    |        |    5. Paste token & click [Test]   |
| 7. Auto-refresh: [HIJACK_BLOCKED]  |        | 6. Alert: [🚨 Hijacking Detected!]  |
| 8. Refresh page -> Kicked to Login |        |    Session Revoked on Server       |
+------------------------------------+        +------------------------------------+
```

1. **Victim Login (Browser A - e.g., Chrome):**
   - Open `http://127.0.0.1:5050` and log in with `student` / `student123`.
   - View the **Protected Dashboard** showing your browser fingerprint and generated token.
   - Click the **"Copy Token"** button in the Demo Controls card.

2. **Simulate Token Theft (Browser B - e.g., Firefox, Edge, or a different device):**
   - Open `http://127.0.0.1:5050/demo/test-session` in Browser B.
   - Paste the stolen token into the textarea and click **"Test Session"**.

3. **Detection & Instant Invalidation:**
   - **Browser B** displays: `🚨 Hijacking Detected!` with a side-by-side comparison table highlighting the mismatched User-Agent and hardware attributes.
   - **Browser A** automatically receives the `HIJACK_BLOCKED` event in the live Server Event Log.
   - When Browser A is refreshed, the victim is immediately redirected to `/login` because the compromised token was revoked on the server.

4. **Baseline Verification (False-Positive Check):**
   - Log in again in Browser A.
   - Open `http://127.0.0.1:5050/demo/test-session` in a new tab in the **same browser**.
   - Paste the token and test: it shows `✅ Fingerprint Matched` (no false alarm).

---

## 📁 Repository Structure & Documentation Index

```
session-hijacking-detection-demo/
├── app.py                            # Flask server (Authentication, Fingerprint comparator, Memory stores)
├── requirements.txt                  # Python dependencies (Flask >= 3.0.0)
├── README.md                         # Quickstart & project overview
├── static/
│   └── style.css                     # Responsive dashboard styling & color-coded badges
├── templates/
│   ├── login.html                    # Login page with silent client-side JS fingerprint harvester
│   ├── dashboard.html                # Protected dashboard with live polling event feed & controls
│   ├── test_session.html             # Attacker simulation page with side-by-side comparison diff
│   └── blocked.html                  # Access blocked notification page
└── docs/
    ├── TEACHER_DEMO_WALKTHROUGH.md   # 🎓 Comprehensive Viva & Presentation guide (Every button & script)
    ├── SYSTEM_ARCHITECTURE.md        # 🏛️ Full technical architecture, Mermaid diagrams, STRIDE threat model
    ├── HOW_IT_WORKS.md               # ⚙️ Code-level breakdown of routes, data structures & algorithms
    └── DEMO_WALKTHROUGH.md           # 🧪 Condensed step-by-step testing instructions
```

### 📚 Detailed Documentation Links
- **[Teacher / Evaluator Walkthrough & Viva Guide (docs/TEACHER_DEMO_WALKTHROUGH.md)](docs/TEACHER_DEMO_WALKTHROUGH.md)**  
  *Detailed walkthrough of every UI element, 5-minute presentation script, and anticipated examiner questions with answers.*
- **[System Architecture & Data Flow (docs/SYSTEM_ARCHITECTURE.md)](docs/SYSTEM_ARCHITECTURE.md)**  
  *Detailed problem statement, block diagrams, Mermaid sequence flows, and production roadmap.*
- **[Technical Deep Dive (docs/HOW_IT_WORKS.md)](docs/HOW_IT_WORKS.md)**  
  *Internal structures (`SESSIONS`, `EVENT_LOG`), algorithms, and file-by-file explanations.*

---

## 🔧 Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| `Address already in use: 5050` | Port 5050 is occupied by another process. | Change port in `app.py` line 284: `app.run(..., port=5051)` or kill the process using port 5050. |
| `Activate.ps1 cannot be loaded` | PowerShell Script Execution Policy restriction. | Run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` or execute directly with `.\venv\Scripts\python.exe app.py`. |
| `ModuleNotFoundError: No module named 'flask'` | Flask not installed in active environment. | Run `pip install -r requirements.txt` (or ensure virtual environment is activated). |

---

## 📄 License
This project is developed for educational and academic demonstration purposes.
