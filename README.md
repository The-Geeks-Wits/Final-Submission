# THE WITS GEEKS FINAL DOCUMENTATION
--
Group Members:

-Makovha Thendo – 2585063 | 2585063@students.wits.ac.za

-Ozuko Mabongo – 2768437 | 2768437@students.wits.ac.za

-Musawenkosi Nzuza – 2560516 | 2560516@students.wits.ac.za

-Tlhalefo Sebaeng – 2575338 | 2575338@students.wits.ac.za

-Dikeledi Mokoatle – 2769106 | 2769106@students.wits.ac.za

## Content:
1. Overview of The System
2. System Architecture
3. Setup & Installation
4. Authentication & User Management
5. Core System Features
6. Technical Implementations
7. System Security
8. Testing & Code Coverage
9. Deployment
10. Known Issues



## 1. Overview of The System

SA Learnerships is a comprehensive online platform that connects South African work‑seekers with SETA‑accredited learnerships, internships, and apprenticeships. It is a platform that facilitates opportunity listings, applicant profile management, application tracking, notifications, and analytics reporting. The backend is built with Node.js, Express, and MongoDB, providing RESTful API endpoints for the frontend to interact with, and integrates Google OAuth for secure authentication. The frontend is built with HTML, CSS, and JavaScript, delivering a responsive, easy‑to‑navigate interface that dynamically populates standardized qualification, skill, institution, and location data aligned with the South African National Qualifications Framework.

---


## 2. System Architecture

The platform is built using a lightweight client‑server architecture, with a plain JavaScript frontend and a Node.js/Express backend, backed by MongoDB. The specific technology stacks are outlined below.

**Frontend**
- Vanilla HTML5, CSS3, and modular JavaScript (ES modules) for a fast, dependency‑light user interface
- Chart.js for interactive bar charts in the analytics dashboard
- Dynamic data fetching via the Fetch API with credential‑based sessions
- Custom responsive layout without heavy frameworks, styled with a consistent design system

**Backend**
- Node.js with the Express.js framework
- MongoDB as the database, accessed through the Mongoose ODM
- Passport.js with the Google OAuth 2.0 strategy for third‑party authentication
- JSON Web Tokens (JWT) stored in httpOnly cookies for session management
- Multer for handling CV file uploads
- Aggregation pipelines for custom analytics and reporting
- Standardised reference data (qualifications, skills, institutions, locations) loaded from JSON files at server startup

**External Services**
- Google OAuth 2.0 for user identity verification
- Azure App Service for hosting both the static frontend and the backend API
- MongoDB Atlas (or Azure Cosmos DB with MongoDB API) for managed database hosting
- Chart.js CDN for client‑side chart rendering

The system follows a client‑server architecture. The browser loads the static frontend from the Azure App Service and sends REST requests to the backend API endpoints, carrying the user’s JWT cookie for authentication. The server receives these requests, executes the corresponding business logic (including complex aggregations), interacts with the MongoDB database through Mongoose, and returns JSON responses. The frontend then dynamically renders the data for the user, creating a seamless, interactive experience.
---


## 3. System Setup & Installation

**Prerequisites**
Node.js (v16.0.0 or higher)

MongoDB (v5.0 or higher, or a MongoDB Atlas connection string)

npm (comes with Node.js)

A Google Cloud Console project with OAuth 2.0 credentials (for Google login)

```markdown
## System Setup & Installation

### 1. Clone the Repository
Open a terminal in the folder where you want to store the project and run:

```bash
git clone https://github.com/The-Geeks-Wits/sa-learnerships.git
cd sa-learnerships
```

### 2. Install Backend Dependencies
Move into the backend folder and install all required packages:

```bash
cd backend
npm install
cd ..
```

### 3. Configure the Environment Files

#### Frontend Configuration
Create a file named **`env.config.js`** in the **root** of the project with the following content:

```javascript
export const backendURL = () => {
    const environment = 'dev';

    if (environment === 'dev') {
        return 'http://localhost:3000';
    } 
    else if (environment === 'prod') {
        return 'https://sa-learnerships.onrender.com';
    }
};
```
> Make sure `environment = 'dev'` for local testing.

#### Backend Environment Variables
Create a file named **`.env`** inside the `backend` folder with these variables (replace placeholders with your own keys):

```env
SERVER_PORT=3000
DB_URI=mongodb://127.0.0.1:27017/sa_learnerships
CLIENT_URL=http://localhost:5500
API_URL=http://localhost:3000
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
JWT_SECRET=your_jwt_secret
```

### 4. Start MongoDB
Make sure MongoDB is installed and running locally on the default port (`27017`).

### 5. Start the Backend Server
Inside the `backend` folder, run:

```bash
node app.js
```

The server will start on **http://localhost:3000**.

### 6. Run the Frontend
The frontend is built with plain HTML, CSS, and JavaScript. You can serve it using any static server – for example, the **Live Server** extension in VS Code, or by running:

```bash
npx serve .
```
from the project root. The frontend will be available at **http://localhost:5500**.

### 7. Access the Application
Open your browser and visit:

```
http://localhost:5500
```

### Google OAuth Notes
Ensure the following redirect URI is registered in your Google Cloud Console:

```
http://localhost:3000/api/users/google/callback
```
```



---


## 4. Authentication & User Management

The platform supports both manual registration and Google OAuth 2.0 for user authentication, using Passport.js and JSON Web Tokens. A signed JWT is stored in an httpOnly cookie, automatically sent with every API request to secure all protected routes.

**Authentication Flow**
- User initiates login or clicks “Login with Google”.
- For Google OAuth, Passport.js handles the callback, extracting the user’s profile information. New users are automatically created; existing users are recognised and logged in.
- Manual registration validates password strength and stores a bcrypt‑hashed password.
- On success, a signed JWT is placed in a secure httpOnly cookie with the same‑site flag, and the user is redirected to the role‑appropriate dashboard.


**User Roles & Permissions**

**Applicants**
- Create and edit a detailed profile, including NQF‑aligned qualifications, standardised skills, and location.
- Upload a CV for potential employers.
- Browse approved opportunities across various sectors.
- Apply for multiple opportunities with a single click.
- Track the status of each application (Pending, Shortlisted, Rejected) and receive in‑app notifications.

**Providers**
- Post learnership, internship, and apprenticeship opportunities with full details.
- View, manage, and edit their own listings.
- Review all applications received for their postings.
- Shortlist or reject applicants, triggering automatic notifications.
- Access the analytics dashboard, including custom report building and CSV export.

**Admins**
- Approve or reject new opportunity listings submitted by providers.
- Manage all user accounts (view, update role/status, soft‑delete).
- Monitor platform activity through a comprehensive analytics suite.
- Generate custom, exportable reports on application volume, placement rates, and more.
- Oversee the overall health of the platform from a dedicated control panel.

**Session management**

SA Learnerships uses a stateless authentication model with JSON Web Tokens stored in httpOnly cookies. When a user logs in (manually or via Google OAuth), the server signs a JWT containing the user’s email and unique ID, and places it in a secure cookie with the `httpOnly`, `sameSite`, and (in production) `secure` flags. This cookie is automatically included in every subsequent request – there are no server‑side sessions, so the server stays light and scalable. On each protected request, a small middleware reads the cookie, verifies the token, and attaches the decoded user to `req.user`. The code below (from `app.js`) shows the JWT signing inside the Google OAuth strategy; the cookie is set by the route handlers (e.g., `login` or the Google callback) after the token is created.

```javascript
// Inside the Google OAuth strategy in app.js
async (accessToken, refreshToken, profile, done) => {
    let user = await User.findOne({ email: profile.emails[0].value });

    if (!user) {
        user = await User.create({
            firstName: profile.name.givenName || 'Google',
            lastName: profile.name.familyName || 'User',
            email: profile.emails[0].value,
            googleId: profile.id,
            signupMethod: 'google',
        });
    }

    // Generate a stateless session token
    const token = jwt.sign(
        { email: user.email, userId: user._id },
        process.env.JWT_SECRET,
        { expiresIn: '24h' }
    );

    user.token = token;   
    return done(null, user);
}
```

The token is later placed in an httpOnly cookie by the route handler (e.g., in `controller.login` or the Google callback), making the entire session management stateless, secure, and easy to scale.
---


## 5. Core System Features

The SA Learnerships platform offers a full suite of tools for work‑seekers, training providers, and administrators. At its heart are opportunity listings, rich applicant profiles, a streamlined application workflow, real‑time notifications, and powerful analytics.

**Opportunity Listings**
- Providers can post learnerships, internships, and apprenticeships with a title, description, stipend, location, duration, requirements, and a closing date.
- Each listing starts in a **Pending** state and is reviewed by an Admin, who can **Approve** it (making it publicly visible) or **Reject** it.
- All approved opportunities are searchable by applicants through a dedicated opportunities page.

**Applicant Profiles**
- Applicants create and maintain a detailed profile containing:
  - **Personal details** – first name, last name, email, gender, date of birth, phone, and a standardised location (selected from a province → city dropdown).
  - **NQF‑aligned qualifications** – a dynamic education form where users pick a qualification level (e.g., Bachelor’s Degree, Diploma) and see the corresponding NQF level automatically. The dropdown data is sourced from official SAQA‑aligned JSON files.
  - **Standardised skills** – a two‑level category → skill selector built from a curated list of ten recognised skill categories, ensuring clean, searchable data.
  - **CV upload** – applicants can upload their CV (PDF, DOC, DOCX), which is stored securely on the server.
- Duplicate qualifications and skills are prevented both on the frontend and through backend validation.

**Application Workflow**
- Applicants can browse and apply for multiple opportunities with a single click, provided their profile is complete.
- Providers view all applications for their postings in one place. They can **shortlist** or **reject** candidates, changing the application’s status accordingly.
- Each status change triggers an in‑app notification for the applicant, keeping them informed of their progress.

**Notifications**
- A real‑time notification badge appears in the navigation bar, showing the number of unread notifications for the logged‑in user.
- Notifications are generated for application status updates (Pending → Shortlisted, Shortlisted → Rejected) and new matching opportunities.
- Users can visit a dedicated notifications page to see a full list.

**Analytics & Reporting**
The platform provides three dashboard reports for Admins and Providers, all exportable as CSV:
- **Application Volume per Opportunity** – a bar chart and table showing how many applications each opportunity has received.
- **Placement Success Rates by Sector** – aggregated placement data broken down by sector.
- **Custom View** – a flexible report builder where the user selects:
  - Dimensions (e.g., Location, NQF Level)
  - Metric (e.g., Shortlisting Rate, Total Applications)
  - Optional filters (status, date range)
  The system runs a MongoDB aggregation across applications, users, and opportunities, then displays the results as a bar chart and table. The report can be downloaded with a timestamped filename.

**Authentication & User Roles**
- Users can register and log in manually or use **Google OAuth 2.0** for quick sign‑up.
- Three distinct roles – **Applicant**, **Provider**, and **Admin** – each have role‑specific views and permissions:
  - Applicants can manage their profile and track applications.
  - Providers can create opportunities, view/manage applications, and access analytics.
  - Admins can moderate listings, manage users, and view the full analytics suite.
- Authentication is handled via a `jwt` cookie (httpOnly), which is automatically sent with every API request.

All these features are built on top of a shared **SA Data Integration** layer that loads official qualification types, institutions (grouped by type), skills by category, and locations by province at server startup, ensuring every piece of profile and opportunity data aligns with South African standards.

---


## 6. Technical Implementations

**Database Models**

The system uses three main Mongoose models:

- **User** – stores profile information, qualifications, skills, and CV file path.
- **Opportunity** – represents learnerships, internships, or apprenticeships posted by providers.
- **Application** – links an applicant to an opportunity and tracks its status (Pending, Shortlisted, Rejected).

**JWT Authentication**

After login, a signed JSON Web Token (JWT) is placed in an httpOnly cookie. This cookie is sent with every request automatically.

```javascript
// from controller.js (register/login)
const token = utils.generateAccessToken(email, user._id);
res.cookie('jwt', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: process.env.NODE_ENV === 'production' ? 'None' : 'Lax',
    maxAge: rememberMe ? 604800000 : 3600000,
});
```

**Protected Routes**

Every protected endpoint uses middleware that reads the JWT from the cookie and verifies it. If the token is missing or invalid, the request is rejected.

```javascript
// from routes.js (cvAuth middleware)
const token = req.headers.authorization || req.cookies?.jwt;
if (!token) return res.status(401).json({ error: 'No Token Provided' });
const decoded = jwt.verify(token, process.env.JWT_SECRET);
req.user = { email: decoded.email, userId: decoded.userId || decoded.id };
```

**Profile Editing**

The `editProfile` function updates all personal fields, qualifications, and skills in one go. It does not enforce strict SAQA validation in this version for flexibility.

```javascript
// from controller.js (editProfile)
const updateOptions = {
    firstName: req.body.firstName, lastName: req.body.lastName,
    email: req.body.email, gender: req.body.gender,
    location: req.body.location, phone: req.body.phone,
    qualifications: req.body.qualifications,
    skills: req.body.skills,
};
const user = await User.findByIdAndUpdate(req.user._id, updateOptions, { new: true });
```

**CV Upload**

Users can upload a CV via Multer. Old files are deleted automatically, and the new file path is saved.

```javascript
// from controller.js (uploadCV )
const filePath = `/uploads/${req.file.filename}`;
if (user.cv) fs.unlinkSync(path.join(process.cwd(), user.cv));
await User.findByIdAndUpdate(req.user.userId, { cv: filePath });
```

**Custom Analytics Report**

The analytics module uses a MongoDB aggregation pipeline that joins Applications with Users, groups data by chosen dimensions, and calculates metrics like shortlist rate.

```javascript
// from analyticsController.js 
pipeline.push(
    { $lookup: { from: 'users', localField: 'applicant', foreignField: '_id', as: 'applicantData' } },
    { $unwind: '$applicantData' },
    { $group: { _id: { location: '$applicantData.location', nqfLevel: '$maxNqf' }, totalApplications: { $sum: 1 } } }
);
const results = await Application.aggregate(pipeline);
```

**Main API Endpoints**

| Area | Method | Endpoint | Purpose |
|------|--------|----------|---------|
| Auth | POST | `/api/users/register` | Create account |
| Auth | POST | `/api/users/login` | Login, receive JWT cookie |
| Profile | GET/PUT | `/api/users/profile` | View/update own profile |
| CV | POST | `/api/users/upload-cv` | Upload CV file |
| Data | GET | `/api/users/data/qualifications` | SAQA-aligned qualifications |
| Data | GET | `/api/users/data/institutions` | Institutions grouped by type |
| Data | GET | `/api/users/data/skills` | Standardised skills by category |
| Data | GET | `/api/users/data/locations` | Provinces and cities |
| Analytics | POST | `/api/analytics/custom-report` | Generate a custom report |
| Admin | GET/PUT/DELETE | `/api/users` and `/api/users/:id` | Manage users (admin only) |

All endpoints return JSON. Protected routes require a valid JWT cookie; admin‑only routes also check the user’s role.

---

## 7. System Security

Our platform uses several layers of protection:

**Authentication**
- Manual sign‑up with **bcrypt‑hashed** passwords, or **Google OAuth 2.0** for password‑less login.
- The Google strategy requests only profile and email; no sensitive Google data is stored.

**Session Handling**
- On login, a signed **JWT** (email + user ID, expires in 24 h) is placed in an **httpOnly**, **same‑site** cookie.  
  ```javascript
  res.cookie('jwt', token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: process.env.NODE_ENV === 'production' ? 'None' : 'Lax',
      maxAge: 3600000,
  });
  ```
- No server‑side sessions – the cookie itself holds the session.

**Route Protection**
- A `cvAuth` middleware checks the cookie (or `Authorization` header) on every protected route, returning **401** if the token is missing or invalid.  
  ```javascript
  const token = req.headers.authorization || req.cookies?.jwt;
  if (!token) return res.status(401).json({ error: 'No Token Provided' });
  jwt.verify(token, process.env.JWT_SECRET, …);
  ```
- Admin‑only endpoints also check the user’s role and return **403** if not admin.

**Role‑Based Access**
- Three roles – **Applicant**, **Provider**, **Admin** – are enforced on both the frontend (sidebar visibility) and backend (middleware).

**CORS**
- The server only accepts requests from the configured `CLIENT_URL`:
  ```javascript
  app.use(cors({ origin: process.env.CLIENT_URL, credentials: true }));
  ```

**File Uploads**
- **Multer** assigns a unique timestamped name to each CV file, preventing collisions. The uploads folder is served only after authentication.
**Password Requirements**
- Manual passwords must be at least 8 characters with uppercase, lowercase, digits, and special symbols. They are never stored in plain text.
---
## 8. Testing & Code Coverage
The primary goal of testing was to ensure that all controllers, services, and routes behave as expected with little to no bugs
The testing process focused on:
- Verifying correct functionality
- Validating error handling 
- Improving overall code coverage

Jest was used as the testing framework for unit testing and mocking.

The screenshot below shows the code coverage results generated after running the automated test suite using the testing framework. The report provides a summary of statement coverage, branch coverage, function coverage, and line coverage across the application. The high coverage percentages indicate that most parts of the system, including validation logic, error handling, conditional branches, and successful execution paths, were tested thoroughly. This demonstrates that the implemented unit tests effectively verified the reliability and correctness of the backend functionality.

<img width="1130" height="428" alt="Screenshot 2026-05-22 162216" src="https://github.com/user-attachments/assets/8f7cbff2-6ea5-447f-b6d7-1971f3c296f6" />

---
## 9. Deployment
**Render & Firebase Hosting**

The application backend is deployed on Render, and the frontend is hosted on Firebase Hosting.
The live application can be accessed at: https://salearnership-734cd.web.app/

---

## 10. Known Issues

* **CSV export uses semicolons** – Some location names contain commas (like “Johannesburg, Gauteng”). To keep the spreadsheet from breaking, the download uses semicolons instead of commas. A few spreadsheet programs may ask you to confirm the separator when you open the file.
* **Location dropdown is built into the code** – The list of provinces and cities is written directly into the profile editor. If a new city needs to be added, a developer must change the code and redeploy the application.
* **Old qualifications may look incomplete** – Before the NQF feature was added, some qualifications were saved without a qualification level or NQF number. Those older entries are still kept in the profile so nothing is lost, but they won’t show the extra NQF information.
* **Charts depend on an internet library** – The bar charts load a small library from the internet. If the internet connection is very slow or the library is blocked, the charts may not appear, but the table and the download button will still work.
* **Analytics reports need sample data** – The custom reports will show “No data found” until sample applications, users, and opportunities are added to the database. A separate script is provided to fill the database with test data so the reports become meaningful.
