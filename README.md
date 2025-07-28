# DeedPro Backend API

## 🏗️ Architecture Overview

**DeedPro uses a two-repository architecture:**

- **Frontend Repository:** [`easydeed/new-front`](https://github.com/easydeed/new-front) - Next.js application
- **Backend Repository:** [`easydeed/deedpro-backend-2024`](https://github.com/easydeed/deedpro-backend-2024) - FastAPI backend (this repo)

## 🚀 What This Backend Does

This FastAPI backend provides:

1. **User Authentication & Management**
   - User registration and login
   - JWT-based authentication
   - Profile management
   - Admin functionality

2. **Subscription Management**
   - Stripe integration for payments
   - Plan limits and usage tracking
   - Webhook handling

3. **Deed Generation System** ⭐
   - HTML deed preview generation
   - PDF deed generation with WeasyPrint
   - Jinja2 template rendering
   - Support for Grant Deeds and Quitclaim Deeds

4. **AI Assistance**
   - OpenAI integration for property assistance
   - Intelligent form completion suggestions

## 🔗 Frontend Integration

### Repository Connection
```
Frontend (new-front)     ←→     Backend (deedpro-backend-2024)
├── Next.js App                 ├── FastAPI Server
├── React Components            ├── Authentication APIs
├── UI/UX Layer                 ├── Deed Generation APIs
└── User Interface              └── Database Layer
```

### Deployment Architecture
```
Frontend: Vercel          ←→     Backend: Render
├── Static Site                 ├── API Server
├── Client-side Routing         ├── Database Connection
└── Environment Variables       └── File Processing
```

### Key API Integration Points

**1. Deed Preview Generation**
```javascript
// Frontend calls this endpoint for deed previews
POST /generate-deed-preview
```

**2. Full Deed Generation**
```javascript
// Frontend calls this endpoint for final PDF generation
POST /generate-deed
```

**3. User Authentication**
```javascript
// Authentication flow
POST /register
POST /login
GET /profile
```

## 📄 Deed Generation System

### How It Works

1. **Frontend Form Submission**
   - User fills out deed form in `new-front` repository
   - Form data is converted to `DeedData` format

2. **Template Processing**
   - Backend receives deed data via API
   - Jinja2 loads appropriate template (`grant_deed.html` or `quitclaim_deed.html`)
   - Template is populated with user data

3. **Output Generation**
   - **Preview Mode:** Returns HTML for browser display
   - **Final Mode:** Generates PDF using WeasyPrint + returns base64

### API Endpoints

#### `/generate-deed-preview`
```json
POST /generate-deed-preview
{
  "deed_type": "grant_deed",
  "data": {
    "grantor": "John Doe",
    "grantee": "Jane Smith",
    "county": "Los Angeles",
    "property_description": "Lot 1, Block 2...",
    // ... other deed fields
  }
}
```

**Response:**
```json
{
  "html": "<html>...rendered deed...</html>",
  "deed_type": "grant_deed",
  "status": "preview_ready"
}
```

#### `/generate-deed`
```json
POST /generate-deed
{
  "deed_type": "quitclaim_deed",
  "data": { /* same as preview */ }
}
```

**Response:**
```json
{
  "html": "<html>...rendered deed...</html>",
  "pdf_base64": "JVBERi0xLjQK...",
  "deed_type": "quitclaim_deed",
  "status": "success"
}
```

## 🛠️ Tech Stack

- **Framework:** FastAPI
- **Database:** PostgreSQL (via psycopg2)
- **Authentication:** JWT with python-jose
- **Payment Processing:** Stripe API
- **Template Engine:** Jinja2
- **PDF Generation:** WeasyPrint
- **AI Integration:** OpenAI GPT
- **Deployment:** Render

## 📁 Project Structure

```
deedpro-backend-2024/
├── backend/
│   ├── main.py              # Main FastAPI application
│   ├── auth.py              # Authentication utilities
│   ├── database.py          # Database operations
│   ├── ai_assist.py         # AI assistance features
│   └── requirements.txt     # Python dependencies
├── templates/
│   ├── grant_deed.html      # Grant deed template
│   └── quitclaim_deed.html  # Quitclaim deed template
└── README.md               # This file
```

## 🔧 Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:pass@host:port/db

# Authentication
SECRET_KEY=your_secret_key
ACCESS_TOKEN_EXPIRE_MINUTES=60

# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# OpenAI
OPENAI_API_KEY=sk-...

# CORS (for frontend connection)
FRONTEND_URL=https://your-frontend-domain.com
```

## 🚀 Local Development

1. **Install Dependencies**
   ```bash
   cd backend
   pip install -r requirements.txt
   ```

2. **Set Environment Variables**
   ```bash
   cp .env.example .env
   # Edit .env with your values
   ```

3. **Run the Server**
   ```bash
   uvicorn main:app --reload --port 8000
   ```

4. **Access API Documentation**
   - Swagger UI: `http://localhost:8000/docs`
   - ReDoc: `http://localhost:8000/redoc`

## 🔄 Frontend Communication

### CORS Configuration
The backend is configured to accept requests from the frontend domain:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configured for Vercel deployment
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Frontend Environment Setup
The frontend (`new-front`) connects via:

```javascript
// Frontend environment variable
NEXT_PUBLIC_API_URL=https://your-backend-domain.render.com
```

### Authentication Flow
```
Frontend                    Backend
   │                          │
   ├─ POST /register ────────→ │ Create user
   ├─ POST /login ───────────→ │ Return JWT token
   ├─ Store JWT in localStorage │
   ├─ Include JWT in headers ─→ │ Verify token
   └─ Access protected routes ─→ │ Return user data
```

## 📋 Deed Template System

### Template Structure
Templates use Jinja2 syntax for dynamic content:

```html
<p>{{ grantor or "_____________________" }}</p>
<p>hereby GRANT(S) to</p>
<p>{{ grantee or "_____________________" }}</p>
```

### Supported Deed Types
- `grant_deed` → `templates/grant_deed.html`
- `quitclaim_deed` → `templates/quitclaim_deed.html`

### CSS Styling
Templates include print-optimized CSS:
- 8.5" x 11" page size
- Times New Roman font
- Legal document formatting
- Signature and notary sections

## 🔒 Security Features

- **JWT Authentication:** Secure token-based auth
- **Password Hashing:** bcrypt with salt rounds
- **CORS Protection:** Controlled cross-origin requests
- **Input Validation:** Pydantic models for data validation
- **Environment Variables:** Sensitive data protection

## 📊 Database Schema

Key tables:
- `users` - User accounts and profiles
- `deeds` - Generated deed records
- `subscriptions` - Stripe subscription data
- `usage_tracking` - Plan limit enforcement

## 🚨 Error Handling

The API returns structured error responses:

```json
{
  "detail": "Deed generation failed: Template not found"
}
```

Common HTTP status codes:
- `200` - Success
- `400` - Bad Request (invalid data)
- `401` - Unauthorized (invalid/missing token)
- `403` - Forbidden (insufficient permissions)
- `500` - Internal Server Error

## 📈 Monitoring & Logging

- FastAPI automatic documentation
- Structured error logging
- Performance monitoring via Render
- Database connection health checks

## 🔄 Deployment

**Current Deployment:** Render.com
- Automatic deploys from `main` branch
- Environment variables configured in Render dashboard
- PostgreSQL database addon connected
- SSL/HTTPS automatically configured

**Frontend Deployment:** Vercel (separate repository)
- Deploys from `easydeed/new-front` repository
- Environment variable `NEXT_PUBLIC_API_URL` points to this backend

## 🤝 Contributing

1. Create feature branch from `main`
2. Make changes in `/backend/` directory only
3. Test locally with frontend integration
4. Push to this repository (`deedpro-backend-2024`)
5. Render automatically deploys on merge to `main`

## 📞 Support

For technical issues:
- Check API documentation at `/docs`
- Review error logs in Render dashboard
- Ensure frontend environment variables are correct
- Verify database connection status

---

**Last Updated:** January 2025  
**Repository:** https://github.com/easydeed/deedpro-backend-2024  
**Frontend Repository:** https://github.com/easydeed/new-front 