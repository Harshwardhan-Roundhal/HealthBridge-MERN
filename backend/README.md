# HealthBridge Backend - Rebuild Guide

A step-by-step guide to recreate the Node.js/Express backend from scratch.

---

## 📋 Prerequisites

- Node.js installed
- MongoDB database (local or Atlas)
- Cloudinary account (for image uploads)
- Stripe/Razorpay accounts (for payments)

---

## 🚀 Step-by-Step Build Guide

### Step 1: Initialize Project

```bash
mkdir backend && cd backend
npm init -y
```

**Why:** Sets up package.json for dependency management.

---

### Step 2: Install Dependencies

```bash
npm install express mongoose dotenv cors bcrypt jsonwebtoken validator multer cloudinary razorpay stripe
npm install -D nodemon
```

**Why:**
- `express`: Web framework for REST API
- `mongoose`: MongoDB ODM for schema modeling
- `dotenv`: Loads environment variables from .env
- `cors`: Enables cross-origin requests from frontend
- `bcrypt`: Hashes passwords securely
- `jsonwebtoken`: Creates JWT tokens for authentication
- `validator`: Validates email/password format
- `multer`: Handles file uploads (images)
- `cloudinary`: Cloud image storage service
- `razorpay/stripe`: Payment gateway integration

---

### Step 3: Configure package.json

Add to package.json:
```json
{
  "type": "module",
  "scripts": {
    "start": "node server.js",
    "server": "nodemon server.js"
  }
}
```

**Why:** `"type": "module"` enables ES6 imports. Scripts provide easy start commands.

---

### Step 4: Create .env File

Create `.env` in backend root:
```
PORT=4000
MONGODB_URI=mongodb://localhost:27017/healthbridge
JWT_SECRET=your-secret-key-here
CLOUDINARY_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_SECRET_KEY=your-secret-key
STRIPE_SECRET_KEY=your-stripe-secret
RAZORPAY_KEY_ID=your-razorpay-key
RAZORPAY_KEY_SECRET=your-razorpay-secret
CURRENCY=INR
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=admin123
```

**Why:** Centralizes configuration. Never commit .env to git (add to .gitignore).

---

### Step 5: Database Configuration

Create `config/mongodb.js`:
```javascript
import mongoose from "mongoose";

const connectDB = async () => {
    mongoose.connection.on('connected', () => console.log("Database Connected"))
    await mongoose.connect(`${process.env.MONGODB_URI}`)
}

export default connectDB;
```

**Why:** Separates DB connection logic. Event listener confirms successful connection.

---

### Step 6: Cloudinary Configuration

Create `config/cloudinary.js`:
```javascript
import { v2 as cloudinary } from 'cloudinary';

const connectCloudinary = async () => {
    cloudinary.config({
        cloud_name: process.env.CLOUDINARY_NAME,
        api_key: process.env.CLOUDINARY_API_KEY,
        api_secret: process.env.CLOUDINARY_SECRET_KEY
    });
}

export default connectCloudinary;
```

**Why:** Centralizes Cloudinary setup. Called once on server start.

---

### Step 7: Initialize Basic Server

Create `server.js`:
```javascript
import express from "express"
import cors from 'cors'
import 'dotenv/config'
import connectDB from "./config/mongodb.js"
import connectCloudinary from "./config/cloudinary.js"

const app = express()
const port = process.env.PORT || 4000

// Initialize connections
connectDB()
connectCloudinary()

// Middlewares
app.use(express.json()) // Parse JSON bodies
app.use(cors()) // Enable CORS

// Test route
app.get("/", (req, res) => {
  res.send("API Working - Server is running!")
})

app.listen(port, () => console.log(`Server started on PORT:${port}`))
```

**Why:** 
- Create server early to test basic setup and connections
- `express.json()` parses incoming JSON (required for POST/PUT)
- `cors()` allows frontend to make requests from different origin
- Test route confirms server is running

#### 🧪 **Test Checkpoint 1: Server & Connections**

```bash
npm run server
```

**Expected Output:**
- Console: "Server started on PORT:4000"
- Console: "Database Connected" (if MongoDB running)
- Browser/Postman: `GET http://localhost:4000/` → "API Working - Server is running!"

**If errors:**
- Check MongoDB is running: `mongod` or verify Atlas connection string
- Verify all .env variables are set correctly

---

### Step 8: Create Models

#### 8.1 User Model (`models/userModel.js`)
```javascript
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    image: { type: String, default: 'default-image-url' },
    phone: { type: String, default: '000000000' },
    address: { type: Object, default: { line1: '', line2: '' } },
    gender: { type: String, default: 'Not Selected' },
    dob: { type: String, default: 'Not Selected' },
})

const userModel = mongoose.models.user || mongoose.model("user", userSchema);
export default userModel;
```

**Why:** 
- `unique: true` prevents duplicate emails
- `mongoose.models.user` prevents re-registration in dev/hot-reload
- Default values allow partial updates

#### 8.2 Doctor Model (`models/doctorModel.js`)
```javascript
const doctorSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    image: { type: String, required: true },
    speciality: { type: String, required: true },
    degree: { type: String, required: true },
    experience: { type: String, required: true },
    about: { type: String, required: true },
    fees: { type: Number, required: true },
    available: { type: Boolean, default: true },
    slots_booked: { type: Object, default: {} },
    address: { type: Object, required: true },
    date: { type: Number, required: true },
}, { minimize: false })
```

**Why:**
- `slots_booked: Object` stores `{ "2024-01-15": ["10:00", "11:00"] }` for booking logic
- `minimize: false` preserves empty objects (Mongoose removes them by default)
- `date: Number` stores timestamp for sorting/querying

#### 8.3 Appointment Model (`models/appointmentModel.js`)
```javascript
const appointmentSchema = new mongoose.Schema({
    userId: { type: String, required: true },
    docId: { type: String, required: true },
    slotDate: { type: String, required: true },
    slotTime: { type: String, required: true },
    userData: { type: Object, required: true },
    docData: { type: Object, required: true },
    amount: { type: Number, required: true },
    date: { type: Number, required: true },
    cancelled: { type: Boolean, default: false },
    payment: { type: Boolean, default: false },
    isCompleted: { type: Boolean, default: false }
})
```

**Why:**
- Denormalized `userData` and `docData` prevent join queries and improve read performance
- Status flags (`cancelled`, `payment`, `isCompleted`) enable flexible filtering

#### 🧪 **Test Checkpoint 2: Models**

Create a test file `test-models.js`:
```javascript
import mongoose from "mongoose"
import 'dotenv/config'
import userModel from './models/userModel.js'

// Test connection and model
mongoose.connect(process.env.MONGODB_URI).then(() => {
    console.log("Connected to DB")
    
    // Test user model
    const testUser = new userModel({
        name: "Test User",
        email: "test@test.com",
        password: "test123"
    })
    
    testUser.save().then(() => {
        console.log("✅ User model works!")
        mongoose.connection.close()
    })
})
```

```bash
node test-models.js
```

**Expected:** "Connected to DB" → "✅ User model works!"

**Why Test Now:** Verifies schemas are correct before building controllers.

---

### Step 9: Create Middleware

#### 9.1 Multer (`middleware/multer.js`)
```javascript
import multer from "multer";

const storage = multer.diskStorage({
    filename: function (req, file, callback) {
        callback(null, file.originalname)
    }
});

const upload = multer({ storage: storage })
export default upload;
```

**Why:** Handles multipart/form-data for image uploads. Stores files temporarily before Cloudinary upload.

#### 9.2 User Auth (`middleware/authUser.js`)
```javascript
import jwt from 'jsonwebtoken'

const authUser = async (req, res, next) => {
    const { token } = req.headers
    if (!token) {
        return res.json({ success: false, message: 'Not Authorized Login Again' })
    }
    try {
        const token_decode = jwt.verify(token, process.env.JWT_SECRET)
        req.body.userId = token_decode.id
        next()
    } catch (error) {
        res.json({ success: false, message: error.message })
    }
}

export default authUser;
```

**Why:**
- Extracts `userId` from JWT and attaches to `req.body` for controller use
- Protects routes without repeating auth logic in each controller

#### 9.3 Admin Auth (`middleware/authAdmin.js`)
```javascript
const authAdmin = async (req, res, next) => {
    const { atoken } = req.headers
    const token_decode = jwt.verify(atoken, process.env.JWT_SECRET)
    if (token_decode !== process.env.ADMIN_EMAIL + process.env.ADMIN_PASSWORD) {
        return res.json({ success: false, message: 'Not Authorized' })
    }
    next()
}
```

**Why:** Different header name (`atoken` vs `token`) separates admin/user auth. Simple string check for single admin account.

#### 9.4 Doctor Auth (`middleware/authDoctor.js`)
```javascript
const authDoctor = async (req, res, next) => {
    const { dtoken } = req.headers
    const token_decode = jwt.verify(dtoken, process.env.JWT_SECRET)
    req.body.docId = token_decode.id
    next()
}
```

**Why:** Similar to user auth but uses `dtoken` header and extracts `docId`.

#### 🧪 **Test Checkpoint 3: JWT Middleware**

Test JWT token generation and verification:
```javascript
// test-jwt.js
import jwt from 'jsonwebtoken'
import 'dotenv/config'

// Generate token (like login)
const token = jwt.sign({ id: "test123" }, process.env.JWT_SECRET)
console.log("Generated token:", token)

// Verify token (like middleware)
const decoded = jwt.verify(token, process.env.JWT_SECRET)
console.log("✅ JWT works! Decoded:", decoded)
```

```bash
node test-jwt.js
```

**Expected:** Token generated and verified successfully.

**Why Test Now:** Auth middleware depends on JWT working correctly.

---

### Step 10: Create Controllers

#### 10.1 User Controller (`controllers/userController.js`)

**Registration:**
```javascript
const registerUser = async (req, res) => {
    const { name, email, password } = req.body;
    
    // Validation
    if (!validator.isEmail(email)) {
        return res.json({ success: false, message: "Invalid email" })
    }
    
    // Hash password
    const salt = await bcrypt.genSalt(10);
    const hashedPassword = await bcrypt.hash(password, salt)
    
    // Create user & return JWT
    const newUser = new userModel({ name, email, password: hashedPassword })
    const user = await newUser.save()
    const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET)
    
    res.json({ success: true, token })
}
```

**Why:**
- `bcrypt.genSalt(10)` adds security (10 rounds = good balance of speed/security)
- JWT returned immediately allows frontend to auto-login user

**Login:**
```javascript
const loginUser = async (req, res) => {
    const { email, password } = req.body;
    const user = await userModel.findOne({ email })
    
    const isMatch = await bcrypt.compare(password, user.password)
    if (isMatch) {
        const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET)
        res.json({ success: true, token })
    }
}
```

**Why:** `bcrypt.compare()` securely checks password against hash.

**Book Appointment:**
```javascript
const bookAppointment = async (req, res) => {
    const { userId, docId, slotDate, slotTime } = req.body
    const docData = await doctorModel.findById(docId)
    
    // Check slot availability
    let slots_booked = docData.slots_booked
    if (slots_booked[slotDate]?.includes(slotTime)) {
        return res.json({ success: false, message: 'Slot Not Available' })
    }
    
    // Update doctor's booked slots
    slots_booked[slotDate] = slots_booked[slotDate] || []
    slots_booked[slotDate].push(slotTime)
    await doctorModel.findByIdAndUpdate(docId, { slots_booked })
    
    // Create appointment
    const newAppointment = new appointmentModel({ userId, docId, slotDate, slotTime, ... })
    await newAppointment.save()
}
```

**Why:** 
- Checks availability before booking (prevents double-booking)
- Updates doctor's `slots_booked` object atomically
- Appointment stores denormalized data for quick reads

**Payment (Razorpay):**
```javascript
const paymentRazorpay = async (req, res) => {
    const { appointmentId } = req.body
    const appointmentData = await appointmentModel.findById(appointmentId)
    
    const options = {
        amount: appointmentData.amount * 100, // Convert to paise
        currency: process.env.CURRENCY,
        receipt: appointmentId,
    }
    
    const order = await razorpayInstance.orders.create(options)
    res.json({ success: true, order })
}
```

**Why:** Razorpay requires amount in smallest currency unit (paise for INR). Order ID used for verification later.

#### 10.2 Admin Controller (`controllers/adminController.js`)

**Add Doctor:**
```javascript
const addDoctor = async (req, res) => {
    const { name, email, password, ... } = req.body
    const imageFile = req.file // From multer
    
    // Upload image to Cloudinary
    const imageUpload = await cloudinary.uploader.upload(imageFile.path)
    const imageUrl = imageUpload.secure_url
    
    // Hash password & create doctor
    const hashedPassword = await bcrypt.hash(password, 10)
    const newDoctor = new doctorModel({ name, email, password: hashedPassword, image: imageUrl, ... })
    await newDoctor.save()
}
```

**Why:** Cloudinary upload happens before saving to DB. If upload fails, doctor isn't created.

#### 10.3 Doctor Controller (`controllers/doctorController.js`)

Similar patterns: login, get appointments, cancel/complete appointments, dashboard stats.

**Key Functions:**
- `loginDoctor`: Authenticates doctor, returns JWT token
- `appointmentsDoctor`: Gets all appointments for logged-in doctor
- `doctorDashboard`: Calculates earnings, patient count, appointment stats
- `changeAvailablity`: Toggles doctor's availability status

**Why:** Doctor needs to manage their own appointments and see dashboard stats.

---

### Step 11: Create Routes

#### 11.1 User Routes (`routes/userRoute.js`)
```javascript
import express from 'express';
import { loginUser, registerUser, getProfile, ... } from '../controllers/userController.js';
import upload from '../middleware/multer.js';
import authUser from '../middleware/authUser.js';

const userRouter = express.Router();

userRouter.post("/register", registerUser)
userRouter.post("/login", loginUser)
userRouter.get("/get-profile", authUser, getProfile)
userRouter.post("/update-profile", upload.single('image'), authUser, updateProfile)
userRouter.post("/book-appointment", authUser, bookAppointment)

export default userRouter;
```

**Why:**
- `upload.single('image')` processes file before controller
- Middleware order matters: `authUser` runs before controller
- Separates routing logic from business logic

#### 11.2 Admin Routes (`routes/adminRoute.js`)
```javascript
import express from 'express';
import { loginAdmin, addDoctor, appointmentsAdmin, ... } from '../controllers/adminController.js';
import authAdmin from '../middleware/authAdmin.js';
import upload from '../middleware/multer.js';

const adminRouter = express.Router();

adminRouter.post("/login", loginAdmin)
adminRouter.post("/add-doctor", authAdmin, upload.single('image'), addDoctor)
adminRouter.get("/appointments", authAdmin, appointmentsAdmin)
adminRouter.get("/dashboard", authAdmin, adminDashboard)

export default adminRouter;
```

#### 11.3 Doctor Routes (`routes/doctorRoute.js`)
```javascript
import express from 'express';
import { loginDoctor, appointmentsDoctor, ... } from '../controllers/doctorController.js';
import authDoctor from '../middleware/authDoctor.js';

const doctorRouter = express.Router();

doctorRouter.post("/login", loginDoctor)
doctorRouter.get("/appointments", authDoctor, appointmentsDoctor)
doctorRouter.get("/list", doctorList) // Public route for frontend
doctorRouter.get("/dashboard", authDoctor, doctorDashboard)

export default doctorRouter;
```

**Why:**
- Admin routes use `atoken` header (via authAdmin middleware)
- Doctor routes use `dtoken` header (via authDoctor middleware)
- `doctor/list` is public (no auth) - frontend needs to display all doctors
- Other routes are protected with respective auth middleware

#### 🧪 **Test Checkpoint 4: Routes & Controllers**

Update `server.js` to include routes:
```javascript
// Add after middlewares, before app.listen
import userRouter from "./routes/userRoute.js"
import adminRouter from "./routes/adminRoute.js"
import doctorRouter from "./routes/doctorRoute.js"

app.use("/api/user", userRouter)
app.use("/api/admin", adminRouter)
app.use("/api/doctor", doctorRouter)
```

Test basic endpoint:
```bash
# Start server
npm run server

# Test registration (Postman or curl)
POST http://localhost:4000/api/user/register
Body: {
  "name": "John Doe",
  "email": "john@test.com",
  "password": "password123"
}

# Expected: { "success": true, "token": "..." }
```

**Why Test Now:** Verifies routes are connected and controllers work before building frontend.

---

### Step 12: Complete Server Setup

Final `server.js` should now have all routes integrated:

```javascript
import express from "express"
import cors from 'cors'
import 'dotenv/config'
import connectDB from "./config/mongodb.js"
import connectCloudinary from "./config/cloudinary.js"
import userRouter from "./routes/userRoute.js"
import doctorRouter from "./routes/doctorRoute.js"
import adminRouter from "./routes/adminRoute.js"

const app = express()
const port = process.env.PORT || 4000

// Initialize connections
connectDB()
connectCloudinary()

// Middlewares
app.use(express.json()) // Parse JSON bodies
app.use(cors()) // Enable CORS

// Routes
app.use("/api/user", userRouter)
app.use("/api/admin", adminRouter)
app.use("/api/doctor", doctorRouter)

app.get("/", (req, res) => res.send("API Working"))

app.listen(port, () => console.log(`Server started on PORT:${port}`))
```

**Final Architecture:**
- All routes mounted with `/api/{role}` prefix
- Middleware processes requests in order: CORS → JSON parsing → Route → Auth → Controller
- Database and Cloudinary initialized at server start

---

## 🧪 Complete Testing Guide

### Test Flow 1: User Registration & Login

**Step 1:** Register a new user
```bash
POST http://localhost:4000/api/user/register
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@test.com",
  "password": "password123"
}

# Expected: { "success": true, "token": "eyJhbGc..." }
```

**Step 2:** Login with same user
```bash
POST http://localhost:4000/api/user/login
Content-Type: application/json

{
  "email": "john@test.com",
  "password": "password123"
}

# Expected: { "success": true, "token": "eyJhbGc..." }
```

**Step 3:** Get profile (protected route)
```bash
GET http://localhost:4000/api/user/get-profile
Headers: {
  "token": "paste-token-from-step-1-or-2"
}

# Expected: { "success": true, "userData": { "name": "John Doe", ... } }
```

**Why This Order:** Registration creates user → Login verifies credentials → Protected route tests JWT auth.

---

### Test Flow 2: Admin Operations

**Step 1:** Admin login
```bash
POST http://localhost:4000/api/admin/login
Content-Type: application/json

{
  "email": "admin@example.com",
  "password": "admin123"
}

# Expected: { "success": true, "token": "..." }
# Save this token as "atoken"
```

**Step 2:** Get all appointments (admin only)
```bash
GET http://localhost:4000/api/admin/appointments
Headers: {
  "atoken": "paste-admin-token-here"
}

# Expected: { "success": true, "appointments": [...] }
```

**Why:** Verifies admin auth middleware works with `atoken` header.

---

### Test Flow 3: Doctor Operations

**Step 1:** Get doctor list (public endpoint)
```bash
GET http://localhost:4000/api/doctor/list

# Expected: { "success": true, "doctors": [...] }
```

**Step 2:** Login as doctor (if you have doctor in DB)
```bash
POST http://localhost:4000/api/doctor/login
Content-Type: application/json

{
  "email": "doctor@test.com",
  "password": "doctor123"
}

# Expected: { "success": true, "token": "..." }
# Save as "dtoken"
```

**Why:** Public list endpoint allows frontend to show doctors without auth. Login uses `dtoken` header.

---

### Test Flow 4: Appointment Booking (Full Flow)

**Prerequisites:** User logged in (have user token), Doctor exists in DB

**Step 1:** Book appointment
```bash
POST http://localhost:4000/api/user/book-appointment
Headers: {
  "token": "user-token-here"
}
Content-Type: application/json

{
  "docId": "doctor-id-from-db",
  "slotDate": "2024-01-15",
  "slotTime": "10:00"
}

# Expected: { "success": true, "message": "Appointment Booked" }
```

**Step 2:** Get user's appointments
```bash
GET http://localhost:4000/api/user/appointments
Headers: {
  "token": "user-token-here"
}

# Expected: { "success": true, "appointments": [...] }
```

**Why:** Tests complete booking flow - slot availability check, doctor update, appointment creation.

---

### Test Flow 5: Payment Integration (Razorpay)

**Step 1:** Create payment order
```bash
POST http://localhost:4000/api/user/payment-razorpay
Headers: {
  "token": "user-token-here"
}
Content-Type: application/json

{
  "appointmentId": "appointment-id-from-previous-test"
}

# Expected: { "success": true, "order": { "id": "...", "amount": ... } }
```

**Why:** Tests payment gateway integration. In production, frontend uses this order ID to complete payment.

---

## 🐛 Common Issues & Solutions

**Issue:** "Database Connected" but models don't work
- **Solution:** Check MongoDB connection string in .env. For Atlas, ensure IP is whitelisted.

**Issue:** JWT token "Not Authorized" errors
- **Solution:** Verify `JWT_SECRET` in .env matches token generation. Check header name matches (`token` vs `atoken` vs `dtoken`).

**Issue:** Multer file upload errors
- **Solution:** Ensure `req.file` exists. Check FormData is sent with `multipart/form-data` content type.

**Issue:** Cloudinary upload fails
- **Solution:** Verify all three Cloudinary credentials in .env. Check image file path is correct.

**Issue:** Slot booking creates duplicates
- **Solution:** Check slot availability logic in `bookAppointment`. Race conditions can occur - consider transactions for production.

---

## 🏗️ Architecture Decisions

1. **Three Separate Routers:** User, Admin, Doctor - Clean separation of concerns
2. **JWT in Headers:** Stateless auth, scalable across multiple servers
3. **Denormalized Appointment Data:** Faster reads, no joins needed
4. **Object-based Slot Booking:** Flexible date/time storage without schema changes
5. **Cloudinary Integration:** Offloads image storage, provides CDN automatically
6. **Middleware Pattern:** Reusable auth logic, keeps controllers clean
7. **Environment Variables:** Configuration separate from code, secure

---

## 📁 Final Folder Structure

```
backend/
├── config/
│   ├── mongodb.js
│   └── cloudinary.js
├── controllers/
│   ├── userController.js
│   ├── adminController.js
│   └── doctorController.js
├── middleware/
│   ├── authUser.js
│   ├── authAdmin.js
│   ├── authDoctor.js
│   └── multer.js
├── models/
│   ├── userModel.js
│   ├── doctorModel.js
│   └── appointmentModel.js
├── routes/
│   ├── userRoute.js
│   ├── adminRoute.js
│   └── doctorRoute.js
├── server.js
├── package.json
└── .env
```

---

## ✅ Progressive Checklist

### Setup Phase
- [ ] Project initialized with npm
- [ ] All dependencies installed
- [ ] package.json configured with "type": "module"
- [ ] .env file created with all variables
- [ ] **Test Checkpoint 1:** Server starts, DB connects

### Models Phase
- [ ] User model created
- [ ] Doctor model created
- [ ] Appointment model created
- [ ] **Test Checkpoint 2:** Models save data correctly

### Middleware Phase
- [ ] Multer configured
- [ ] User auth middleware created
- [ ] Admin auth middleware created
- [ ] Doctor auth middleware created
- [ ] **Test Checkpoint 3:** JWT tokens generate and verify

### Controllers Phase
- [ ] User controller functions created
- [ ] Admin controller functions created
- [ ] Doctor controller functions created
- [ ] Payment integrations initialized

### Routes Phase
- [ ] User routes created and connected
- [ ] Admin routes created and connected
- [ ] Doctor routes created and connected
- [ ] **Test Checkpoint 4:** Registration endpoint works

### Final Testing
- [ ] User registration and login work
- [ ] Protected routes require authentication
- [ ] Admin login and operations work
- [ ] Doctor list endpoint works (public)
- [ ] Appointment booking flow works
- [ ] Payment order creation works
- [ ] Image uploads work (test with Postman form-data)

## 📚 Understanding the Project Flow

### Request Flow Diagram

```
Frontend Request
    ↓
Express Server (server.js)
    ↓
CORS Middleware (allows cross-origin)
    ↓
JSON Parser (converts body to object)
    ↓
Route Matching (/api/user/register)
    ↓
Auth Middleware (if protected route)
    ↓
Controller Function (business logic)
    ↓
Model/Database Operation
    ↓
Response sent back
```

### Authentication Flow

**User Registration:**
1. Frontend sends: `{ name, email, password }`
2. Controller validates email format
3. Password hashed with bcrypt
4. User saved to MongoDB
5. JWT token generated with user ID
6. Token returned to frontend
7. Frontend stores token in localStorage

**Protected Route Access:**
1. Frontend sends request with `token` in headers
2. Auth middleware extracts token
3. JWT verified with secret
4. User ID extracted from token
5. User ID attached to `req.body.userId`
6. Controller uses `userId` for operations

### Database Relationships

```
User ──< Appointment >── Doctor
      (userId)    (docId)

Appointment stores:
- userId: Reference to user
- docId: Reference to doctor
- userData: Denormalized user info (for faster reads)
- docData: Denormalized doctor info
- slotDate, slotTime: Booking details
- amount: Payment amount
- Status flags: cancelled, payment, isCompleted
```

**Why Denormalization?** Instead of joining tables on every read, we store user and doctor data directly in appointment. This makes queries faster but requires keeping data in sync if user/doctor info changes.

### Slot Booking Logic

Doctor's `slots_booked` structure:
```javascript
{
  "2024-01-15": ["10:00", "11:00", "14:00"],
  "2024-01-16": ["09:00", "15:00"]
}
```

**Booking Process:**
1. Check if date exists in `slots_booked`
2. Check if time slot already in array
3. If available, add time to array
4. Update doctor document
5. Create appointment with same date/time

**Cancellation Process:**
1. Find appointment by ID
2. Get doctor ID, date, time from appointment
3. Remove time from doctor's `slots_booked[date]` array
4. Mark appointment as cancelled

---

## 🎯 Key Takeaways for Interviews

1. **Separation of Concerns:** Models (data), Controllers (logic), Routes (routing), Middleware (cross-cutting)
2. **JWT Authentication:** Stateless, scalable, works across servers
3. **MongoDB Schema Design:** Denormalization for read performance, unique indexes for data integrity
4. **Error Handling:** Try-catch in controllers, consistent error response format
5. **Security:** Password hashing (bcrypt), JWT secrets, environment variables
6. **File Uploads:** Multer for handling, Cloudinary for storage/CDN
7. **Payment Integration:** Razorpay/Stripe order creation, verification on callback

