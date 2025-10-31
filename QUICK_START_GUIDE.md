# ⚡ Quick Start Guide - Get Your Backend Running in 30 Minutes!

---

## 🎯 Goal
Get your backend server running and connected to your frontend in **30 minutes**!

---

## 📋 Prerequisites

Make sure you have these installed:
```bash
# Check versions
node --version    # Should be v16+ or v18+
npm --version     # Should be v8+
git --version     # Any recent version
docker --version  # Optional but recommended
```

Don't have them? Install:
- **Node.js**: [Download here](https://nodejs.org/) (LTS version)
- **Docker**: [Download here](https://www.docker.com/products/docker-desktop/)
- **Git**: [Download here](https://git-scm.com/)

---

## 🚀 Step-by-Step Setup (30 minutes)

### ⏱️ Minutes 1-5: Create Project

```bash
# 1. Go to your project root folder
cd "C:\Users\TheActivist\Downloads\Compressed\bmtc-realtime-api-master\FINAL_WHERE_IS_MY_BUS\project (1)"

# 2. Create backend folder
mkdir backend
cd backend

# 3. Initialize Node.js project
npm init -y

# 4. You're in! Let's continue...
```

### ⏱️ Minutes 6-10: Install Dependencies

```bash
# Install all required packages
npm install express socket.io pg redis ioredis jsonwebtoken bcryptjs axios dotenv cors helmet morgan express-validator qrcode node-cron compression razorpay twilio

# Install development dependencies
npm install --save-dev nodemon

# Check if installed correctly
npm list express
```

### ⏱️ Minutes 11-15: Create Project Structure

```bash
# Windows PowerShell
mkdir src\config, src\controllers, src\models, src\routes, src\services, src\middleware, src\websocket, src\ml, src\utils
mkdir database\migrations, database\seeds

# OR manually create these folders:
backend/
├── src/
│   ├── config/
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   ├── services/
│   ├── middleware/
│   ├── websocket/
│   ├── ml/
│   └── utils/
├── database/
│   ├── migrations/
│   └── seeds/
└── package.json
```

### ⏱️ Minutes 16-20: Setup Database with Docker

Create `docker-compose.yml` in `backend` folder:

```yaml
version: '3.8'

services:
  postgres:
    image: postgis/postgis:15-3.3
    environment:
      POSTGRES_DB: whereismybus
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password123
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

Start database:
```bash
docker-compose up -d

# Check if running
docker ps
```

### ⏱️ Minutes 21-25: Create Basic Files

**1. Create `.env` file:**
```bash
# In backend folder, create .env
```

Add this content:
```env
NODE_ENV=development
PORT=5000
API_URL=http://localhost:5000

DB_HOST=localhost
DB_PORT=5432
DB_NAME=whereismybus
DB_USER=postgres
DB_PASSWORD=password123

REDIS_HOST=localhost
REDIS_PORT=6379

JWT_SECRET=your_super_secret_jwt_key_change_this_in_production
JWT_REFRESH_SECRET=your_refresh_token_secret_change_this
JWT_EXPIRE=15m
JWT_REFRESH_EXPIRE=7d
```

**2. Create `src/app.js`:**
```javascript
require('dotenv').config();
const express = require('express');
const cors = require('cors');

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// Test route
app.get('/', (req, res) => {
  res.json({ 
    message: '🚌 Where Is My Bus - Backend API',
    status: 'Running',
    timestamp: new Date().toISOString()
  });
});

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'OK', uptime: process.uptime() });
});

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`
╔═══════════════════════════════════════════╗
║  🚌 Where Is My Bus - Backend Server     ║
║  🚀 Server running on port ${PORT}         ║
║  🌍 http://localhost:${PORT}               ║
╚═══════════════════════════════════════════╝
  `);
});
```

**3. Update `package.json` scripts:**
```json
{
  "scripts": {
    "start": "node src/app.js",
    "dev": "nodemon src/app.js"
  }
}
```

### ⏱️ Minutes 26-30: Test & Run!

```bash
# Start server
npm run dev

# You should see:
# ╔═══════════════════════════════════════════╗
# ║  🚌 Where Is My Bus - Backend Server     ║
# ║  🚀 Server running on port 5000          ║
# ╚═══════════════════════════════════════════╝

# Test in browser:
# Open: http://localhost:5000
# You should see: {"message":"🚌 Where Is My Bus - Backend API","status":"Running"}
```

---

## ✅ Verification Checklist

```yaml
☐ Node.js installed (v16+)
☐ Docker running
☐ PostgreSQL container running (port 5432)
☐ Redis container running (port 6379)
☐ Backend server running (port 5000)
☐ http://localhost:5000 showing JSON response
☐ http://localhost:5000/health showing OK
```

---

## 🎯 What's Next?

Now that your basic server is running, you need to:

### Step 1: Add Database Connection (Day 1)

Create `src/config/database.js`:
```javascript
const { Pool } = require('pg');
require('dotenv').config();

const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
});

pool.on('connect', () => {
  console.log('✅ Connected to PostgreSQL');
});

pool.on('error', (err) => {
  console.error('❌ PostgreSQL error:', err);
});

module.exports = {
  query: (text, params) => pool.query(text, params),
  pool,
};
```

Test it in `src/app.js`:
```javascript
const db = require('./config/database');

// Add this route
app.get('/test-db', async (req, res) => {
  try {
    const result = await db.query('SELECT NOW()');
    res.json({ 
      status: 'Database connected!', 
      time: result.rows[0].now 
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Step 2: Create Database Tables (Day 1)

Create `database/schema.sql`:
```sql
-- Users table
CREATE TABLE IF NOT EXISTS users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone_number VARCHAR(10) UNIQUE NOT NULL,
    wallet_balance DECIMAL(10,2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Test insert
INSERT INTO users (phone_number, wallet_balance) 
VALUES ('9876543210', 100.00);
```

Run it:
```bash
# Connect to database
docker exec -it backend-postgres-1 psql -U postgres -d whereismybus

# Copy-paste the SQL above, or:
# In Docker Desktop, open postgres container terminal, then:
psql -U postgres -d whereismybus -f /path/to/schema.sql
```

### Step 3: Add Authentication (Day 2-3)

Use the code from `BACKEND_STARTER_CODE.md`:
- Copy `src/controllers/authController.js`
- Copy `src/middleware/auth.js`
- Copy `src/routes/auth.js`

### Step 4: Add Real-time Tracking (Day 4-5)

Add WebSocket support:
```bash
npm install socket.io
```

Update `src/app.js` to include Socket.IO (check `BACKEND_STARTER_CODE.md`)

### Step 5: Connect Frontend (Day 6-7)

Update your frontend API calls:

In `src/services/api.ts`:
```typescript
const API_URL = 'http://localhost:5000/api';

// Replace mock data with real API calls
export const authApi = {
  sendOTP: (phoneNumber: string) => 
    axios.post(`${API_URL}/auth/send-otp`, { phoneNumber }),
    
  verifyOTP: (phoneNumber: string, otp: string) =>
    axios.post(`${API_URL}/auth/verify-otp`, { phoneNumber, otp }),
};
```

---

## 🔥 Quick Commands Reference

```bash
# Start everything
docker-compose up -d        # Start database
npm run dev                 # Start backend

# Stop everything
Ctrl+C                      # Stop backend
docker-compose down         # Stop database

# View logs
docker logs backend-postgres-1  # PostgreSQL logs
docker logs backend-redis-1     # Redis logs

# Connect to database
docker exec -it backend-postgres-1 psql -U postgres -d whereismybus

# Useful SQL commands
\dt                         # List all tables
\d users                    # Describe users table
SELECT * FROM users;        # Query users
\q                          # Quit
```

---

## 🆘 Troubleshooting

### Problem: "Port 5000 already in use"
```bash
# Windows: Find and kill process
netstat -ano | findstr :5000
taskkill /PID <PID> /F

# Or change port in .env
PORT=5001
```

### Problem: "Cannot connect to PostgreSQL"
```bash
# Check if container is running
docker ps

# Restart container
docker-compose restart postgres

# Check logs
docker logs backend-postgres-1
```

### Problem: "Module not found"
```bash
# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

### Problem: "Redis connection failed"
```bash
# Check if running
docker ps | grep redis

# Restart
docker-compose restart redis
```

---

## 📚 Next Steps After Setup

1. ✅ **Week 1**: Complete authentication system
   - Follow `BACKEND_STARTER_CODE.md` → Auth section

2. ✅ **Week 2**: Add bus tracking APIs
   - Follow `BACKEND_STARTER_CODE.md` → Bus Controller section

3. ✅ **Week 3**: Payment integration
   - Get Razorpay account (free)
   - Follow `BACKEND_STARTER_CODE.md` → Payment section

4. ✅ **Week 4**: Real-time WebSocket
   - Follow `BACKEND_STARTER_CODE.md` → WebSocket section

5. ✅ **Week 5-6**: ML predictions
   - Follow `BACKEND_STARTER_CODE.md` → ML Service section

---

## 🎯 Success Criteria

You'll know you're successful when:

```yaml
✅ Server starts without errors
✅ Can access http://localhost:5000
✅ Database connection works
✅ Redis connection works
✅ Can create API routes
✅ Frontend can call backend APIs
✅ Real-time updates work
```

---

## 📞 Resources

- **Full Backend Code**: See `BACKEND_STARTER_CODE.md`
- **Architecture**: See `BACKEND_ARCHITECTURE_PROPOSAL.md`
- **Roadmap**: See `PROJECT_ENHANCEMENT_SUMMARY.md`

---

## 🎉 You're Ready!

You now have:
- ✅ Backend server running
- ✅ Database setup
- ✅ Development environment ready
- ✅ Foundation for all features

**Next**: Start adding features from `BACKEND_STARTER_CODE.md`!

---

*Good luck! You've got this! 💪🚀*

