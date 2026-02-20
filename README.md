# HealthBridge

HealthBridge is a full-stack doctor appointment platform built with the MERN ecosystem.
It includes:
- a patient-facing web app (`frontend`)
- an admin/doctor dashboard (`admin`)
- a REST API backend (`backend`)

## Tech Stack

- Frontend: React, Vite, Tailwind CSS, Axios, React Router
- Admin Panel: React, Vite, Tailwind CSS, Axios, React Router
- Backend: Node.js, Express.js, MongoDB (Mongoose), JWT
- Media and Payments: Cloudinary, Razorpay, Stripe
- Deployment: Vercel

## Project Structure

```text
HealthBridge/
  admin/       # Admin + doctor dashboard
  backend/     # Express API + MongoDB
  frontend/    # Patient-facing app
```

## Core Features

- User registration/login and profile management
- Browse doctors and book appointments
- Appointment cancellation and history
- Doctor dashboard for appointments and profile updates
- Admin dashboard for doctors, appointments, and analytics
- Online payments (Razorpay and Stripe)

## API Base URL

- Local: `http://localhost:4000`
- Production: `https://<your-backend>.vercel.app`

## Main API Routes

### User Routes (`/api/user`)

- `POST /register`
- `POST /login`
- `GET /get-profile` (protected: `token` header)
- `POST /update-profile` (protected: `token`)
- `POST /book-appointment` (protected: `token`)
- `GET /appointments` (protected: `token`)
- `POST /cancel-appointment` (protected: `token`)
- `POST /payment-razorpay` (protected: `token`)
- `POST /verifyRazorpay` (protected: `token`)
- `POST /payment-stripe` (protected: `token`)
- `POST /verifyStripe` (protected: `token`)

### Doctor Routes (`/api/doctor`)

- `POST /login`
- `GET /list` (public)
- `POST /cancel-appointment` (protected: `dtoken` header)
- `GET /appointments` (protected: `dtoken`)
- `POST /change-availability` (protected: `dtoken`)
- `POST /complete-appointment` (protected: `dtoken`)
- `GET /dashboard` (protected: `dtoken`)
- `GET /profile` (protected: `dtoken`)
- `POST /update-profile` (protected: `dtoken`)

### Admin Routes (`/api/admin`)

- `POST /login`
- `POST /add-doctor` (protected: `atoken` header)
- `GET /appointments` (protected: `atoken`)
- `POST /cancel-appointment` (protected: `atoken`)
- `GET /all-doctors` (protected: `atoken`)
- `POST /change-availability` (protected: `atoken`)
- `GET /dashboard` (protected: `atoken`)

## Environment Variables

Create a `.env` file in each app as needed.

### `backend/.env`

```env
PORT=4000
MONGODB_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret

ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=your_admin_password

CLOUDINARY_NAME=your_cloudinary_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_SECRET_KEY=your_cloudinary_secret_key

RAZORPAY_KEY_ID=your_razorpay_key_id
RAZORPAY_KEY_SECRET=your_razorpay_key_secret
STRIPE_SECRET_KEY=your_stripe_secret_key

CURRENCY=INR
```

### `frontend/.env`

```env
VITE_BACKEND_URL=http://localhost:4000
VITE_RAZORPAY_KEY_ID=your_razorpay_key_id
```

### `admin/.env`

```env
VITE_BACKEND_URL=http://localhost:4000
VITE_CURRENCY=INR
```

## Run Locally

### 1. Install dependencies

```bash
cd backend && npm install
cd ../frontend && npm install
cd ../admin && npm install
```

### 2. Start apps (three terminals)

Terminal 1:
```bash
cd backend
npm run server
```

Terminal 2:
```bash
cd frontend
npm run dev
```

Terminal 3:
```bash
cd admin
npm run dev
```

## Deploy on Vercel

- Deploy `backend`, `frontend`, and `admin` as separate Vercel projects.
- Set environment variables in each Vercel project.
- In frontend/admin Vercel env, set `VITE_BACKEND_URL` to the deployed backend URL.
- Ensure backend CORS includes deployed frontend/admin origins (with `https://`).

## Quick Health Check

- Backend root route:
  - `GET https://<your-backend>.vercel.app/`
  - Expected response: `API Working`
- Note: `/api/health` is not defined by default in current backend code.

## Common Issues

- `EADDRINUSE: 4000`: another process is using port 4000. Stop it or change `PORT`.
- Atlas connection error on Vercel: allow Vercel access in MongoDB Atlas Network Access and verify `MONGODB_URI`.
- CORS blocked requests: add exact frontend/admin domains in backend CORS config.

## Security Note

- Never commit real secrets to GitHub (`.env` values, API keys, DB credentials).
- Rotate any credentials that were exposed in development history.
