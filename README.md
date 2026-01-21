# Philip’s Blog 🧛‍♂️💻

A personal blog website built **from scratch** using **Flask**, **Python**, **Bootstrap**, and **PostgreSQL (Supabase)** — deployed on **Render**.


![Blog Homepage Screenshot](static/assets/img/preview.png)

---

## ✨ Features

- 📝 **Full CRUD** for blog posts (create, read, update, delete)
- 🔑 **User authentication** (register, login, logout)
- 🖋️ **Rich text editing** with Flask-CKEditor
- 💬 **Comment system** linked to blog posts
- 👤 **Gravatar integration** for user profile images
- 📱 **Responsive design** using Bootstrap 5
- 🔒 **Admin-only routes** protected via decorators
- 📧 **Resilient contact form** with SMTP email sending (timeout-safe)

---

## 🛠️ Technologies Used

| Category           | Tools                                              |
|--------------------|----------------------------------------------------|
| **Backend**        | Python, Flask, Gunicorn                            |
| **Database**       | PostgreSQL (Supabase), SQLAlchemy ORM              |
| **Frontend**       | Bootstrap 5, Jinja2, Flask-CKEditor                |
| **Authentication** | Flask-Login, Werkzeug                              |
| **Deployment**     | Render, GitHub                                     |
| **Security**       | python-dotenv, environment variables, Supabase RLS |

---

## 🚀 Deployment & Live Demo

The app is deployed on **Render** using **Gunicorn** as the production WSGI server.

🔗 **Live Site:**  
https://philips-blog-9kuj.onrender.com/

> ⚠️ **Note:** Render free-tier services may take ~30–50 seconds to spin up if idle.

---

## 🗄️ Database Migration & “Resurrection” Story

This project went through **three database phases**:

1. **SQLite (Local Development)**
2. **Render PostgreSQL (Initial Deployment)**
3. **Supabase PostgreSQL (Current Production Setup)**

The final migration to **Supabase** significantly improved reliability and persistence.

### Migration Highlights

- Seamless migration by updating only the `DATABASE_URL`
- No application logic changes thanks to SQLAlchemy
- PostgreSQL-compatible schema and data structure
- Supabase **Row Level Security (RLS)** enabled for core tables
- Blog successfully restored after months of downtime — hence the “resurrection” theme

---

## 📧 SMTP Email & 502 Error Fix (Production Issue)

After deployment, the contact form caused intermittent **HTTP 502 errors** on Render.

### Root Cause

- Blocking Gmail SMTP handshakes during slow network responses
- Environment variables loaded at startup instead of request time

### Fix Implemented

- Environment variables are now fetched **inside the email function**
- Added `timeout=15` to the SMTP connection
- Errors are logged safely without crashing the request

Result:  
✅ Contact page stays responsive  
✅ No more 502 errors  
✅ Email failures don’t take down the app

---

## 📦 Local Installation

### 1. Clone the repository

```bash
git clone https://github.com/legophil101/Blog-Website.git
cd Blog-Website
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

http://127.0.0.1:5000

## 📚 Lessons Learned

- Resolving dependency conflicts (Flask, Jinja2, MarkupSafe)
- Configuring PostgreSQL on Render
- Using .env for security
- Keeping Render free-tier sites alive with UptimeRobot/cron-job.org
- Deploying Flask apps with Gunicorn + Render
- Migrating databases without breaking production code
- Debugging deployment-only issues not present locally
- Handling SMTP failures safely in production
- Using environment variables correctly across environments
- Understanding the difference between local SQLite and cloud PostgreSQL
- Reading and acting on Render runtime logs at 4–5 AM ☕

## 🧠 My Journey

This project started as part of my programming course, and it turned into my first fully deployed web app.

Some challenges I overcame:

- Figuring out why my Python version wasn’t compatible on Render (fixed by specifying a compatible Python version in
  Render configuration)
- Handling PostgreSQL database setup and migration
- Debugging Render build logs with persistence and problem-solving
- It took months, late nights, and a lot of trial and error — but hitting “live” for the first time made it all worth
  it.

Better Call Phil for your next Flask site. 😁
