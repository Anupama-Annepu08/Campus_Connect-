# CampusConnect — Lost & Found System
Spring Boot + MySQL + React (all-in-one)

---

## 🚀 How to Run (No Terminal Needed)

### Step 1 — Set up MySQL

1. Open **MySQL Workbench**
2. Connect to your local MySQL server (usually `localhost:3306`)
3. Open a new SQL tab and run this single line:
   ```sql
   CREATE DATABASE IF NOT EXISTS campusconnect;
   ```
4. That's it! Hibernate will auto-create all tables on first startup.

### Step 2 — Configure your MySQL password

Open `backend/src/main/resources/application.properties` and change:
```properties
spring.datasource.password=root
```
Replace `root` with your actual MySQL root password.

### Step 3 — Run the Spring Boot app in IntelliJ

1. Open the `backend` folder in IntelliJ IDEA
2. Wait for Maven to download dependencies
3. Run `BackendApplication.java` (the green ▶ button)
4. The app starts at **http://localhost:8080**

### Step 4 — Create the first Admin account

Since registration only creates students, use MySQL Workbench to insert an admin:

```sql
USE campusconnect;
INSERT INTO users (full_name, email, password, role, college_name, phone_number, created_at)
VALUES ('Admin User', 'admin@campusconnect.com', 'admin123', 'ADMIN', 'Your College', '9999999999', NOW());
```

Or use the pre-built admin at:
- **Email:** `admin@campusconnect.com`
- **Password:** `admin123`

---

## 📁 Project Structure

```
campusconnect/
├── backend/                          ← Spring Boot app (serves everything)
│   ├── pom.xml
│   └── src/main/
│       ├── java/com/campusconnect/backend/
│       │   ├── BackendApplication.java
│       │   ├── config/
│       │   │   ├── CorsConfig.java   ← NEW: global CORS
│       │   │   └── SpaController.java
│       │   ├── model/
│       │   │   ├── User.java         ← FIXED: @JsonIgnore on password
│       │   │   ├── Item.java         ← FIXED: @JsonIgnoreProperties
│       │   │   └── Claim.java        ← FIXED: @JsonIgnoreProperties
│       │   ├── repository/
│       │   ├── service/
│       │   └── controller/
│       └── resources/
│           ├── application.properties ← FIXED: MySQL URL + Jackson config
│           ├── init.sql               ← NEW: MySQL setup script
│           └── static/               ← Built React app goes here
└── frontend/                         ← React source (already built into static/)
    └── src/
        └── pages/
            └── AdminDashboard.js     ← FIXED: setCurrentPage prop added
```

---

## 🔧 All Fixes Applied

| # | File | What was fixed |
|---|------|---------------|
| 1 | `application.properties` | Added `createDatabaseIfNotExist=true`, Jackson config, proper MySQL URL params |
| 2 | `User.java` | Added `@JsonIgnore` on `password` — never sent to frontend |
| 3 | `Item.java` | Added `@JsonIgnoreProperties` on `reportedBy` — prevents JSON recursion crash |
| 4 | `Claim.java` | Added `@JsonIgnoreProperties` on `item` and `claimedBy` — prevents JSON recursion |
| 5 | `CorsConfig.java` | NEW file — global CORS so React dev server can call the API |
| 6 | `AdminDashboard.js` | Added `setCurrentPage` to function props (was declared in App.js but not received) |

---

## 🌐 API Endpoints

### Users
| Method | URL | Description |
|--------|-----|-------------|
| POST | `/api/users/register` | Register student |
| POST | `/api/users/login` | Student login |
| POST | `/api/users/admin/register` | Register admin |
| POST | `/api/users/admin/login` | Admin login |
| GET | `/api/users/{id}` | Get user by ID |
| GET | `/api/users/all` | Get all users |

### Items
| Method | URL | Description |
|--------|-----|-------------|
| POST | `/api/items/report/{userId}` | Report lost/found item |
| GET | `/api/items/approved` | Get all approved items |
| GET | `/api/items/pending` | Get items pending approval |
| GET | `/api/items/all` | Get all items (admin) |
| GET | `/api/items/my/{userId}` | Get user's own items |
| PUT | `/api/items/approve/{id}` | Approve item (admin) |
| PUT | `/api/items/reject/{id}` | Reject item (admin) |
| GET | `/api/items/match?title=&category=` | AI match search |

### Claims
| Method | URL | Description |
|--------|-----|-------------|
| POST | `/api/claims/submit?itemId=&userId=&message=` | Submit a claim |
| GET | `/api/claims/pending` | Get pending claims (admin) |
| GET | `/api/claims/all` | Get all claims (admin) |
| GET | `/api/claims/my/{userId}` | Get user's claims |
| PUT | `/api/claims/approve/{id}` | Approve claim (admin) |
| PUT | `/api/claims/reject/{id}` | Reject claim (admin) |

---

## 🗄️ Database Tables (auto-created by Hibernate)

**users** — id, full_name, email, password, role, college_name, phone_number, created_at

**items** — id, title, description, category, location, type, status, image_path, date_lost_or_found, reported_by (FK → users), created_at

**claims** — id, item_id (FK → items), claimed_by (FK → users), message, status, created_at

---

## 🔄 Item & Claim Status Flow

```
Student reports item → PENDING_APPROVAL
Admin approves       → APPROVED        (visible to all students)
Admin rejects        → REJECTED
Student claims item  → Claim: PENDING
Admin approves claim → Claim: APPROVED  +  Item: RETURNED
Admin rejects claim  → Claim: REJECTED
```
# Campus_Connect-
