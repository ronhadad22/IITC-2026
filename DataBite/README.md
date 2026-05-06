# DataBite — Restaurant Menu Intelligence

A full-stack app for collecting and analyzing restaurant menu data.

- **Frontend:** React + Vite + TailwindCSS
- **Backend:** Python (FastAPI)
- **Database:** None (mock data for now)

---

## Quick Start

### 1. Start the Backend

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

API available at: http://localhost:8000  
Docs at: http://localhost:8000/docs

### 2. Start the Frontend

```bash
cd frontend
npm install
npm run dev
```

App available at: http://localhost:5173

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/stats` | Dashboard statistics |
| GET | `/api/restaurants` | All restaurants |
| GET | `/api/restaurants/:id` | Single restaurant |
| GET | `/api/menu-items` | All menu items (filterable) |
| GET | `/api/menu-items?restaurant=X` | Filter by restaurant |
| GET | `/api/menu-items?category=X` | Filter by category |
