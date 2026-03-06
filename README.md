# Philip’s Blog 🧛‍♂️💻

A personal blog website built **from scratch** using **Flask**, **Python**, **Bootstrap**, and **PostgreSQL (Supabase)** — deployed on **Render**.

![Blog Homepage Screenshot](static/assets/img/preview.png)

---

## ✨ Features

- 📝 **Full CRUD** for blog posts (create, read, update, delete).
- 🔑 **User authentication** (register, login, logout).
- 🖋️ **Rich text editing** with Flask-CKEditor.
- 💬 **Comment system** linked to blog posts.
- 👤 **Gravatar integration** for user profile images.
- 📱 **Responsive design** using Bootstrap 5.
- 🔒 **Admin-only routes** protected via custom decorators.
- 📧 **Resilient contact form** using Formspree API to bypass cloud-provider SMTP port restrictions.
  - Smooth scroll to the header after a successful submission.
  - Hides the form and shows a success message dynamically.
  - Works reliably with Formspree.
- 🛠️ **Tech Highlight**: AJAX form submission with fetch API for smooth UX.

---

## 🛠️ Technologies Used

| Category           | Tools                                              |
|--------------------|----------------------------------------------------|
| **Backend** | Python, Flask, Gunicorn                            |
| **Database** | PostgreSQL (Supabase), SQLAlchemy ORM              |
| **Frontend** | Bootstrap 5, Jinja2, Flask-CKEditor                |
| **Authentication** | Flask-Login, Werkzeug                              |
| **Deployment** | Render, GitHub                                     |
| **Security** | python-dotenv, environment variables, Supabase RLS |

---

## 🚀 Deployment & Live Demo

The app is deployed on **Render** using **Gunicorn** as the production WSGI server.

🔗 **Live Site:** [https://philips-blog-9kuj.onrender.com/](https://philips-blog-9kuj.onrender.com/)

> ⚠️ **Note:** Render free-tier services may take ~30–50 seconds to spin up if idle.

---

## 🗄️ Database Migration & “Resurrection” Story

This project went through **three database phases**:

1. **SQLite (Local Development)**
2. **Render PostgreSQL (Initial Deployment)**
3. **Supabase PostgreSQL (Current Production Setup)**

The final migration to **Supabase** significantly improved reliability and persistence.

### Migration Highlights
- Seamless migration by updating only the `DATABASE_URL`.
- No application logic changes thanks to SQLAlchemy.
- PostgreSQL-compatible schema and data structure.
- Supabase **Row Level Security (RLS)** enabled for core tables.
- Blog successfully restored after months of downtime — hence the “resurrection” theme.

---

## 📧 SMTP Email & 502 Error Fix

After deployment, the contact form caused intermittent **HTTP 502 errors** on Render.

### Root Cause
- Blocking Gmail SMTP handshakes during slow network responses.
- Environment variables loaded at startup instead of request time.

### Fix Implemented
- Environment variables are now fetched **inside the email function**.
- Added `timeout=15` to the SMTP connection.
- Errors are logged safely without crashing the request.

---

## 🏗️ Production Deployment Notes

After deploying on Render, we encountered production-specific challenges that did not appear in local development.

### **The Issue: Gunicorn Timeout Loop**
* **Symptoms:** Logs showed `CRITICAL WORKER TIMEOUT` and `SIGKILL` every ~30 seconds.
* **Problem:** The app crashed before the homepage could load because the default Gunicorn "patience" is only 30 seconds.

### **The Cause**
* On free-tier Render instances, the combination of a "cold-start" and the initial **Supabase** database handshake often takes ~35–40 seconds.
* Gunicorn assumed the app was frozen and killed the process right before it could finish initializing.

### 🛠️ Fixes Implemented

#### **Gunicorn & Worker Optimization**

* **Database Eager Loading (`joinedload`):** Optimized the homepage query to eliminate the "N+1 Problem." By using SQLAlchemy's `joinedload`, the app now fetches posts, authors, and metadata in **one single database trip** instead of 19+, significantly reducing cross-region latency between Singapore (Render) and Mumbai (Supabase).
* **Gunicorn Configuration:** * **Command:** `gunicorn -w 1 --threads 4 --timeout 120 main:app`
    * **Stability over Concurrency:** Tuned to a single worker with multiple threads to maximize available RAM (512MB limit) and prevent **Error 520/SIGKILL** crashes. 
    * **Latency Handling:** Increased threads allow the server to handle secondary tasks while waiting for long-distance database responses.
* **Payload Reduction:** Implemented a `.limit(5)` constraint on the initial homepage load to reduce data transfer size and improve "Time to First Byte" (TTFB) on throttled free-tier connections.

#### **Advanced Database Connection Pooling**

* **Configuration:**
```python
app.config['SQLALCHEMY_ENGINE_OPTIONS'] = {
    "pool_pre_ping": True,
    "pool_recycle": 300,
    "pool_size": 10,
    "max_overflow": 20,
}

```

* **Result:** Banished the `PendingRollbackError` and `OperationalError`. The **pool_pre_ping** ensures the app "tests" the connection before every query, silently reconnecting if the Singapore-to-Mumbai pipe has timed out.

#### **Optimized Startup Logic**

* **Strategy:** Commented out `db.create_all()` inside the `app_context`.
* **Result:** Drastically reduced startup time and prevented Render's "Health Check" from failing. Since tables are already persistent in Supabase, this prevents redundant, slow handshakes on every deploy.

#### **Verified Database URL Protocol**

* **Fix:** Ensured `DATABASE_URL` uses the `postgresql://` prefix instead of the deprecated `postgres://`.
* **Requirement:** Critical for compatibility with **SQLAlchemy 2.0+** and modern Psycopg2 drivers.

#### **Heartbeat Route (`/ping`)**

* **Feature:** A lightweight route that returns a `200 OK` status without querying the database.
* **Integration:** Designed for **UptimeRobot** to keep the service "warm," preventing the 15-minute idle sleep on Render's Free Tier without adding load to the database.

#### **Cascade Delete Logic**

* **Model Update:** `comments = relationship("Comment", back_populates="parent_post", cascade="all, delete")`
* **Result:** Prevents `IntegrityError` when deleting blog posts by automatically cleaning up associated comments, ensuring database referential integrity.

---

### **Current Status**
* ✅ App boots reliably on Render free-tier.
* ✅ Posts and comments handle deletions safely.
* ✅ CKEditor warnings are suppressed and form binding is fixed.
* ✅ Custom 500 error pages provide a professional fallback.

---

## 📦 Local Installation

### 1. Clone the repository
```bash
git clone [https://github.com/legophil101/blog-website.git](https://github.com/legophil101/blog-website.git)
cd blog-website

```

### 2. Create a .env file

```env
SECRET_KEY=your_secret_key
DATABASE_URL=your_supabase_postgres_url
EMAIL_KEY=your_email_address
PASSWORD_KEY=your_16_character_gmail_app_password

```

### 3. Install dependencies

```bash
pip install -r requirements.txt

```

### 4. Run the app

```bash
python main.py

```

### 5. Open in browser

[http://127.0.0.1:5000](http://127.0.0.1:5000)

---

## 📚 Lessons Learned

* Resolving dependency conflicts (Flask, Jinja2, MarkupSafe).
* Configuring PostgreSQL on Render.
* Using `.env` for security.
* Keeping Render free-tier sites alive with UptimeRobot.
* Deploying Flask apps with Gunicorn + Render.
* Migrating databases without breaking production code.
* Debugging deployment-only issues not present locally.
* Handling SMTP failures safely in production.
* Understanding the difference between local SQLite and cloud PostgreSQL.
* Reading and acting on Render runtime logs at 4–5 AM ☕.

---

## 🧠 My Journey

This project started as part of my programming course, and it turned into my first fully deployed web app.

Some challenges I overcame:

* Figuring out why my Python version wasn’t compatible on Render.
* Handling PostgreSQL database setup and migration.
* Debugging Render build logs with persistence and problem-solving.
* It took months, late nights, and a lot of trial and error but hitting “live” for the first time made it all worth it.

Better Call Phil for your next Flask site. 😁
