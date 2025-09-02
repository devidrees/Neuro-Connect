# 🧠 Neuro Connect

A web platform that enables **students**, **doctors**, and **admins** to collaborate for mental-health support. Students create counseling sessions (anonymous optional), doctors manage and respond, and admins verify doctors and monitor platform activity. Includes **real-time chat** and a **community posts** module.

---

## 🚀 Features

- 👩‍🎓 **Student**: create/manage sessions; secure chat with the assigned doctor; interact with posts  
- 🧑‍⚕️ **Doctor**: review/approve sessions; respond and add notes; publish awareness posts  
- 🛡️ **Admin**: verify doctors, list users, view system stats  
- 💬 **Real-time chat** (Socket.io) within each session  
- 🕶️ **Anonymous sessions** with alias for students  
- 📰 **Posts**: doctors share tips/articles; users like/comment  

---

## 🧩 Tech Stack

| Frontend | Backend | Real-Time | Database | Auth | ODM |
|---|---|---|---|---|---|
| React.js | Node.js + Express | Socket.io | MongoDB | JWT + bcrypt | Mongoose |

---

## 📐 Architecture (4-Layer)

- **Presentation**: React SPA (role-aware dashboards, forms, chat UI)  
- **API/Interface**: Express controllers + Socket.io gateway (REST + WebSocket edge)  
- **Application/Domain**: services and policies (RBAC, session status, anonymity)  
- **Data/Infrastructure**: Mongoose models, repositories, adapters (MongoDB)  

---

## 📁 Repository Layout

> GitHub path: **`Neuro-Connect/App/Neuro-Connect-Code`**

# 📁 Repository Layout

```bash
Neuro-Connect/
└─ App/
└─ Neuro-Connect-Code/
├─ frontend/ # React SPA
│ └─ src/...
├─ backend/ # Express + Socket.io
│ ├─ models/ # Mongoose schemas
│ ├─ repositories/ # DB access
│ ├─ services/ # Use-cases
│ ├─ controllers/ # Express controllers
│ ├─ routes/ # REST route definitions
│ ├─ ws/ # Socket.io handlers
│ ├─ middlewares/ # auth.jwt, rbac.guard, validate
│ ├─ config/ # env loader, constants
│ └─ app.js / server.js
├─ docs/ # Diagrams, notes
└─ README.md
```


---

## 🛠️ Setup and Run Locally

### 0) Prerequisites
- **Node.js** ≥ 18 and npm  
- **MongoDB** (local `mongod` or Atlas cluster)

---

### 1) Clone the repo
```bash
git clone https://github.com/<your-org-or-user>/Neuro-Connect.git
cd Neuro-Connect/App/Neuro-Connect-Code
```

## 🛠️ Setup and Run Locally

### 0) Prerequisites
- **Node.js** ≥ 18 and npm  
- **MongoDB** (local `mongod` or Atlas cluster)

---

### 1) Clone the repo
```bash
git clone https://github.com/<your-org-or-user>/Neuro-Connect.git
cd Neuro-Connect/App/Neuro-Connect-Code
```

### 2) Environment Variables

Create a file named .env inside the backend/ folder:

### Server
```bash
PORT=5000
NODE_ENV=development
CLIENT_URL=http://localhost:5173   # must match frontend dev server
```

### Auth
```bash
JWT_SECRET=supersecret_replace_me
JWT_EXPIRES=1d
```

### Database
```bash
MONGO_URI=mongodb://127.0.0.1:27017/neuro_connect
```

Create a file named .env inside the frontend/ folder:
```
VITE_API_BASE_URL=http://localhost:5000
VITE_SOCKET_URL=http://localhost:5000
```

### 3) Install Dependencies

Run npm install separately for backend and frontend:

# Install backend deps
```bash
cd server
npm install
```

# Install frontend deps
```bash
cd ../frontend
npm install
```


### 4) Seed Initial Data

Inside server/, create a script seed.js:
```bash
import mongoose from "mongoose";
import bcrypt from "bcryptjs";
import User from "./models/User.js";
import Session from "./models/Session.js";

const MONGO_URI = process.env.MONGO_URI || "mongodb://127.0.0.1:27017/neuro_connect";

async function runSeed() {
  await mongoose.connect(MONGO_URI);

  // Clear old data
  await User.deleteMany({});
  await Session.deleteMany({});

  // Passwords
  const passwordHash = await bcrypt.hash("password123", 10);

  // Create users
  const admin = await User.create({
    name: "Admin One",
    email: "admin@example.com",
    password: passwordHash,
    role: "admin",
    isActive: true
  });

  const doctor = await User.create({
    name: "Dr. Mira",
    email: "doctor@example.com",
    password: passwordHash,
    role: "doctor",
    specialization: "Psychiatry",
    qualifications: "MBBS, MD",
    experience: 6,
    licenseNumber: "LIC-12345",
    verificationStatus: "approved",
    isActive: true,
    verifiedBy: admin._id,
    verificationDate: new Date()
  });

  const student = await User.create({
    name: "Student One",
    email: "student@example.com",
    password: passwordHash,
    role: "student",
    isActive: true
  });

  // Simulate session booking
  await Session.create({
    student: student._id,
    doctor: doctor._id,
    title: "Exam stress",
    description: "Need help preparing for exams",
    preferredDateTime: new Date(Date.now() + 86400000), // tomorrow
    status: "pending",
    isAnonymous: false,
    chatRoom: "room1"
  });

  console.log("✅ Seed data inserted: admin, doctor, student, and a pending session");
  process.exit();
}

runSeed();
```

Add this script to backend/package.json:
```bash
"scripts": {
  "seed": "node seed.js"
}
```

Run it after starting Mongo:

```bash
cd server
npm run seed
```


### 5) Run the Project

Open two terminals:

Terminal A – Backend

```bash
cd server
npm run dev   # runs Express + Socket.io on port 5000
```

Terminal B – Frontend
```bash
cd frontend
npm run dev   # runs React dev server (usually http://localhost:5173)
```

Now open http://localhost:5173 in your browser.


## 🧱 Role Matrix (essentials)

+----------------------------+---------+--------+-------+
| Action                     | Student | Doctor | Admin |
+----------------------------+---------+--------+-------+
| Create session             |   ✅     |   ❌    |  ❌   |
| Update session status      |   ❌     |   ✅    |  ❌   |
| Chat in assigned session   |   ✅     |   ✅    |  ❌   |
| Create post                |   ❌     |   ✅    |  ❌   |
| Verify doctor              |   ❌     |   ❌    |  ✅   |
| View stats/users           |   ❌     |   ❌    |  ✅   |
+----------------------------+---------+--------+-------+


## 🛡️ Security Notes

Passwords hashed with bcrypt before storage
JWT-based authentication for all protected routes
Role-based access enforced in services and controllers
Environment variables in .env, never commit them
Use HTTPS and secure JWT secrets in production


## 📄 License

This project is under active development. All rights reserved.
Use, distribution, or modification without permission is prohibited.

© 2025 Mohammad Idrees Bhat


## 📬 Contact

Questions or collaboration: dev.idrees@gmail.com


