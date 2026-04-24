# EduMetrics — Full Stack Setup Guide

## Project Structure

```
/
├── backend/          ← Django REST API (from bv.zip)
│   ├── manage.py
│   ├── config/       ← settings, urls, wsgi
│   ├── accounts/     ← JWT auth / advisor login
│   └── analysis_engine/ ← all student risk analysis APIs
│
└── frontend/         ← Static HTML/CSS/JS website
    ├── index.html    ← Landing page + Login form
    ├── dashboard.html ← Main advisor dashboard
    ├── css/
    │   ├── styles.css
    │   └── dashboard-extra.css
    └── js/
        ├── api.js       ← JWT auth + all API calls
        ├── charts.js    ← Chart.js chart builders
        └── script.js    ← App logic (uses live API data)
```

---

## Backend Setup

### 1. Install dependencies

```bash
cd backend
pip install django djangorestframework djangorestframework-simplejwt django-cors-headers python-dotenv django-apscheduler
```

### 2. Create `.env` file inside `backend/`

```env
DJANGO_SECRET_KEY=your-secret-key-here

# Client DB (read-only simulator DB with student data)
CLIENT_ENGINE=django.db.backends.postgresql
CLIENT_NAME=your_client_db_name
CLIENT_HOST=localhost
CLIENT_PORT=5432
CLIENT_USER=your_db_user
CLIENT_PASSWORD=your_db_password

# Internal trigger secret (optional)
INTERNAL_SECRET=your-internal-secret
```

### 3. Run migrations

```bash
python manage.py migrate
```

### 4. Create an advisor account

```bash
python manage.py shell
>>> from accounts.models import Users
>>> u = Users(advisor_id='ADV001', advisor_name='Dr. Priya Malhotra', class_id='CSE_Y1_A')
>>> u.save()
```

> **Password formula:** `{advisor_name}{last 3 digits of advisor_id}`
> For the above: password = `Dr. Priya Malhotra001`

### 5. Start the backend

```bash
python manage.py runserver
```

Backend runs at: **http://localhost:8000**

---

## Frontend Setup

No build step needed — pure HTML/CSS/JS.

### Option A: Open directly in browser

Just open `index.html` in your browser.

> If you see CORS errors, use Option B instead.

### Option B: Serve with a local server (recommended)

```bash
cd frontend
python -m http.server 5173
```

Then visit: **http://localhost:5173**

---

## How Login Works

1. Go to `index.html` → scroll to the login form (or click "Log In" in the nav)
2. Enter your **Advisor ID** (e.g. `ADV001`)
3. Enter your **Password** (`{name}{last3digits}` e.g. `Dr. Priya Malhotra001`)
4. On success → automatically redirected to `dashboard.html`
5. JWT tokens are stored in `localStorage` and auto-refreshed

---

## API Endpoints (all under `/api/analysis/`)

| Endpoint | Description |
|---|---|
| `GET dashboard/summary/` | Stat card data |
| `GET flagged/` | This week's flagged students |
| `GET students/` | Full student roster |
| `GET last_week/` | Prior week comparison |
| `GET interventions/` | Intervention log |
| `GET student/<id>/detail/` | Deep-dive student data |
| `GET analytics/` | Scatter + heatmap data |
| `GET pre_mid_term/` | Midterm predictions |
| `GET risk_of_failing/` | Fail risk scores |

All endpoints require `?class_id=X&semester=Y&sem_week=Z` params.
All endpoints require `Authorization: Bearer <token>` header.

---

## Troubleshooting

**"Could not connect to server"**
→ Make sure the Django backend is running at `localhost:8000`

**CORS errors in browser**
→ Ensure `CORS_ALLOW_ALL_ORIGINS = True` is set in `settings.py` (it is by default)

**Empty dashboard after login**
→ Check that your `class_id` in the advisor's account matches actual data in the client DB

**Token expired errors**
→ The app auto-refreshes tokens. If it keeps happening, log out and back in.
