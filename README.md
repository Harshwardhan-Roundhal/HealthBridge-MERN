# HealthBridge - Vercel Deployment Guide

This guide will help you deploy all three parts of the HealthBridge application (Admin, Frontend, and Backend) on Vercel.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Environment Variables Setup](#environment-variables-setup)
- [Deployment Steps](#deployment-steps)
  - [Backend Deployment](#1-backend-deployment)
  - [Frontend Deployment](#2-frontend-deployment)
  - [Admin Deployment](#3-admin-deployment)
- [Post-Deployment Configuration](#post-deployment-configuration)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before deploying, ensure you have:

1. **Vercel Account**: Sign up at [vercel.com](https://vercel.com)
2. **Vercel CLI** (optional but recommended):
   ```bash
   npm install -g vercel
   ```
3. **GitHub/GitLab/Bitbucket Account**: For connecting repositories
4. **Environment Variables**: MongoDB URI, Cloudinary credentials, JWT secrets, etc.

---

## Project Structure

```
HealthBridge/
├── admin/          # React Admin Panel (Vite)
├── frontend/       # React Frontend App (Vite)
└── backend/        # Node.js/Express Backend API
```

---

## Environment Variables Setup

### Backend Environment Variables

Create a `.env` file in the `backend/` directory with the following variables:

```env
# MongoDB Connection
MONGODB_URI=your_mongodb_atlas_connection_string

# JWT Secrets
JWT_SECRET=your_jwt_secret_key
JWT_EXPIRES_IN=7d

# Cloudinary Configuration
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret

# Payment Gateways (Optional)
RAZORPAY_KEY_ID=your_razorpay_key_id
RAZORPAY_KEY_SECRET=your_razorpay_key_secret
STRIPE_SECRET_KEY=your_stripe_secret_key

# Server Port (Vercel will handle this automatically)
PORT=4000
```

### Frontend Environment Variables

Create a `.env` file in the `frontend/` directory:

```env
VITE_API_URL=https://your-backend-app.vercel.app/api
```

### Admin Environment Variables

Create a `.env` file in the `admin/` directory:

```env
VITE_API_URL=https://your-backend-app.vercel.app/api
```

---

## Deployment Steps

### 1. Backend Deployment

The backend needs to be configured as a serverless function on Vercel.

#### Step 1.1: Create `vercel.json` for Backend

Create a `vercel.json` file in the `backend/` directory:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "server.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "server.js"
    }
  ],
  "env": {
    "NODE_ENV": "production"
  }
}
```

#### Step 1.2: Update `server.js` for Vercel

Ensure your `server.js` exports the Express app for Vercel:

```javascript
// Your existing server.js should already work, but ensure it exports the app:
export default app; // or module.exports = app;
```

**Note**: Since your backend uses ES modules (`"type": "module"`), make sure `server.js` exports the app correctly.

#### Step 1.3: Deploy Backend

**Option A: Using Vercel Dashboard**

1. Go to [vercel.com](https://vercel.com) and sign in
2. Click **"Add New Project"**
3. Import your Git repository
4. Set the **Root Directory** to `backend`
5. Configure build settings:
   - **Framework Preset**: Other
   - **Build Command**: Leave empty (or `npm install`)
   - **Output Directory**: Leave empty
   - **Install Command**: `npm install`
6. Add all environment variables from your `.env` file
7. Click **"Deploy"**

**Option B: Using Vercel CLI**

```bash
cd backend
vercel login
vercel
```

Follow the prompts:
- Set up and deploy? **Yes**
- Which scope? Select your account
- Link to existing project? **No**
- Project name? `healthbridge-backend` (or your preferred name)
- Directory? `./`
- Override settings? **No**

After deployment, add environment variables:

```bash
vercel env add MONGODB_URI
vercel env add JWT_SECRET
vercel env add CLOUDINARY_CLOUD_NAME
vercel env add CLOUDINARY_API_KEY
vercel env add CLOUDINARY_API_SECRET
# Add all other environment variables
```

#### Step 1.4: Get Backend URL

After deployment, note your backend URL (e.g., `https://healthbridge-backend.vercel.app`). You'll need this for frontend and admin configurations.

---

### 2. Frontend Deployment

#### Step 2.1: Update API URL

Before deploying, update your frontend code to use the backend URL. Check your API configuration files (usually in `src/config/` or `src/utils/`) and ensure they use the environment variable:

```javascript
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:4000/api';
```

#### Step 2.2: Deploy Frontend

**Option A: Using Vercel Dashboard**

1. Go to [vercel.com](https://vercel.com)
2. Click **"Add New Project"**
3. Import your Git repository (same repo as backend)
4. Set the **Root Directory** to `frontend`
5. Configure build settings:
   - **Framework Preset**: Vite
   - **Build Command**: `npm run build`
   - **Output Directory**: `dist`
   - **Install Command**: `npm install`
6. Add environment variable:
   - `VITE_API_URL` = `https://your-backend-app.vercel.app/api`
7. Click **"Deploy"**

**Option B: Using Vercel CLI**

```bash
cd frontend
vercel
```

Follow the prompts and set:
- Root Directory: `frontend`
- Build Command: `npm run build`
- Output Directory: `dist`

Add environment variable:

```bash
vercel env add VITE_API_URL
# Enter: https://your-backend-app.vercel.app/api
```

---

### 3. Admin Deployment

#### Step 3.1: Update API URL

Similar to frontend, ensure your admin panel uses the environment variable for the API URL.

#### Step 3.2: Deploy Admin

**Option A: Using Vercel Dashboard**

1. Go to [vercel.com](https://vercel.com)
2. Click **"Add New Project"**
3. Import your Git repository (same repo)
4. Set the **Root Directory** to `admin`
5. Configure build settings:
   - **Framework Preset**: Vite
   - **Build Command**: `npm run build`
   - **Output Directory**: `dist`
   - **Install Command**: `npm install`
6. Add environment variable:
   - `VITE_API_URL` = `https://your-backend-app.vercel.app/api`
7. Click **"Deploy"**

**Option B: Using Vercel CLI**

```bash
cd admin
vercel
```

Follow the prompts and set:
- Root Directory: `admin`
- Build Command: `npm run build`
- Output Directory: `dist`

Add environment variable:

```bash
vercel env add VITE_API_URL
# Enter: https://your-backend-app.vercel.app/api
```

---

## Post-Deployment Configuration

### 1. Update CORS Settings

Ensure your backend `server.js` allows requests from your frontend and admin domains:

```javascript
app.use(cors({
  origin: [
    'https://your-frontend-app.vercel.app',
    'https://your-admin-app.vercel.app',
    'http://localhost:5173', // For local development
    'http://localhost:5174'  // If admin runs on different port
  ],
  credentials: true
}))
```

### 2. MongoDB Atlas Whitelist

Add your Vercel deployment IPs to MongoDB Atlas:
1. Go to MongoDB Atlas → Network Access
2. Click **"Add IP Address"**
3. Click **"Allow Access from Anywhere"** (or add specific Vercel IPs)

### 3. Cloudinary Settings

Ensure your Cloudinary account allows uploads from your Vercel domain.

### 4. Test Your Deployments

1. **Backend**: Visit `https://your-backend-app.vercel.app/` - Should show "API Working"
2. **Frontend**: Visit `https://your-frontend-app.vercel.app`
3. **Admin**: Visit `https://your-admin-app.vercel.app`

---

## Troubleshooting

### Backend Issues

**Problem**: Backend returns 404 or doesn't work
- **Solution**: Ensure `vercel.json` is correctly configured and `server.js` exports the app

**Problem**: Environment variables not working
- **Solution**: Re-add environment variables in Vercel dashboard and redeploy

**Problem**: MongoDB connection fails
- **Solution**: Check MongoDB Atlas network access and connection string

### Frontend/Admin Issues

**Problem**: API calls fail with CORS errors
- **Solution**: Update backend CORS settings to include frontend/admin URLs

**Problem**: Environment variables not available
- **Solution**: Ensure variables start with `VITE_` prefix and rebuild

**Problem**: Build fails
- **Solution**: Check build logs in Vercel dashboard for specific errors

### General Issues

**Problem**: Changes not reflecting after deployment
- **Solution**: Clear Vercel cache and redeploy, or wait a few minutes

**Problem**: Multiple projects in one repository
- **Solution**: Use Vercel's monorepo support or deploy each as separate projects with different root directories

---

## Quick Deploy Commands

### Deploy All Three Projects

```bash
# Backend
cd backend && vercel --prod

# Frontend
cd ../frontend && vercel --prod

# Admin
cd ../admin && vercel --prod
```

### Update Environment Variables

```bash
# Backend
cd backend
vercel env pull .env.local

# Frontend
cd ../frontend
vercel env pull .env.local

# Admin
cd ../admin
vercel env pull .env.local
```

---

## Project URLs

After deployment, you'll have three URLs:

- **Backend API**: `https://healthbridge-backend.vercel.app`
- **Frontend**: `https://healthbridge-frontend.vercel.app`
- **Admin Panel**: `https://healthbridge-admin.vercel.app`

Update these URLs in your environment variables and CORS settings accordingly.

---

## Additional Resources

- [Vercel Documentation](https://vercel.com/docs)
- [Vercel CLI Reference](https://vercel.com/docs/cli)
- [Deploying Express.js on Vercel](https://vercel.com/docs/functions/serverless-functions/runtimes/node-js)
- [Environment Variables in Vercel](https://vercel.com/docs/environment-variables)

---

## Support

If you encounter any issues during deployment, check:
1. Vercel deployment logs
2. Browser console for frontend errors
3. Network tab for API call failures
4. MongoDB Atlas logs
5. Cloudinary dashboard for upload issues

---

**Happy Deploying! 🚀**
