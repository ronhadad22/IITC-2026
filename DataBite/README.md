# DataBite — Restaurant Menu Intelligence

A full-stack app for collecting and analyzing restaurant menu data.

- **Frontend:** React + Vite + TailwindCSS v4
- **Backend:** Python (FastAPI)
- **Database:** None (mock data for now)
- **Deployment:** AWS CloudFormation (VPC, EC2, S3 + CloudFront)

---

## Prerequisites

### Backend
- Python 3.11+

### Frontend
- Node.js 20.19+ or 22.12+ (required by TailwindCSS v4)
- npm 10+

---

## Backend Setup

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

- **App:** http://localhost:8000
- **Swagger Docs:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc

### API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/stats` | Dashboard statistics |
| GET | `/api/restaurants` | All restaurants |
| GET | `/api/restaurants/:id` | Single restaurant |
| GET | `/api/menu-items` | All menu items (filterable) |
| GET | `/api/menu-items?restaurant=X` | Filter by restaurant |
| GET | `/api/menu-items?category=X` | Filter by category |

---

## Frontend Setup

> **Note:** Requires Node.js 20.19+. Check with `node -v`.

```bash
cd frontend
npm install
npm run dev
```

- **App:** http://localhost:5173

The frontend connects to the backend at `http://localhost:8000`. Make sure the backend is running first.

### Features
- Dashboard with live stats
- Restaurant cards (click to filter menu items)
- Menu items table with restaurant filter chips
- Backend connection status indicator (green/red badge in header)

---

## AWS Deployment

CloudFormation templates are in the `../CloudFormation/` directory.

### Option 1: Single EC2 (simple)

Everything on one EC2 instance — Nginx serves frontend + proxies API to uvicorn.

```bash
cd ../CloudFormation

# 1. Create VPC
aws cloudformation deploy --template-file vpc.yaml \
  --stack-name databite-vpc --parameter-overrides EnvironmentName=databite \
  --region us-east-1

# 2. Deploy app on EC2
aws cloudformation deploy --template-file app.yaml \
  --stack-name databite-app \
  --parameter-overrides EnvironmentName=databite KeyName=YOUR_KEY \
  --region us-east-1

# 3. Get the URL
aws cloudformation describe-stacks --stack-name databite-app --region us-east-1 \
  --query "Stacks[0].Outputs[?OutputKey=='AppURL'].OutputValue" --output text
```

### Option 2: CloudFront + S3 + EC2 (production)

Frontend on S3 (served via CloudFront CDN), backend on EC2.

```bash
cd ../CloudFormation

# 1. Deploy (VPC must exist first)
aws cloudformation deploy --template-file cloudfront-app.yaml \
  --stack-name databite-cloudfront \
  --parameter-overrides EnvironmentName=databite KeyName=YOUR_KEY \
  --region us-east-1

# 2. Upload frontend to S3
BUCKET=$(aws cloudformation describe-stacks --stack-name databite-cloudfront \
  --region us-east-1 --query "Stacks[0].Outputs[?OutputKey=='S3BucketName'].OutputValue" --output text)
aws s3 cp frontend/index.html s3://$BUCKET/index.html --content-type text/html --region us-east-1

# 3. Get CloudFront URL
aws cloudformation describe-stacks --stack-name databite-cloudfront --region us-east-1 \
  --query "Stacks[0].Outputs[?OutputKey=='CloudFrontURL'].OutputValue" --output text
```

### Cleanup

```bash
aws cloudformation delete-stack --stack-name databite-cloudfront --region us-east-1
aws cloudformation delete-stack --stack-name databite-app --region us-east-1
aws cloudformation wait stack-delete-complete --stack-name databite-app --region us-east-1
aws cloudformation delete-stack --stack-name databite-vpc --region us-east-1
```

---

## Project Structure

```
DataBite/
├── backend/
│   ├── main.py              # FastAPI app with mock data
│   └── requirements.txt     # fastapi, uvicorn
├── frontend/
│   ├── src/
│   │   ├── App.jsx          # React app (3 tabs: dashboard, restaurants, menu)
│   │   ├── index.css        # TailwindCSS import
│   │   └── main.jsx         # Entry point
│   ├── index.html           # Vite HTML template
│   ├── vite.config.js       # Vite + Tailwind plugin
│   └── package.json
└── README.md

CloudFormation/
├── vpc.yaml                 # VPC with 2 public, 2 app, 2 data subnets
├── app.yaml                 # Single EC2 deployment
├── cloudfront-app.yaml      # CloudFront + S3 + EC2 deployment
└── frontend/
    └── index.html           # Static HTML for S3 (no Node.js needed)
```
