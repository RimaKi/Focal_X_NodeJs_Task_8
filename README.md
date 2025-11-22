# Project Structure:

/project-root
├─ /src
│ ├─ /config
│ │ └─ .env
│ ├─ /controllers
│ │ ├─ auth.controller.js
│ │ ├─ user.controller.js
│ │ ├─ project.controller.js
│ │ ├─ ProjectMember.controller.js
│ │ ├─ task.controller.js
│ │ ├─ LogTask.controller.js
│ │ └─ notification.controller.js
│ ├─ /models
│ │ ├─ user.js
│ │ ├─ project.js
│ │ ├─ ProjectMember.js
│ │ ├─ task.js
│ │ ├─ LogTask.js
│ │ └─ notification.js
│ ├─ /routes
│ │ ├─ auth.routes.js
│ │ ├─ user.routes.js
│ │ ├─ project.routes.js
│ │ ├─ ProjectMember.routes.js
│ │ ├─ task.routes.js
│ │ ├─ LogTask.routes.js
│ │ └─ notification.routes.js
│ ├─ /middleware
│ │ ├─ auth.middleware.js
│ │ ├─ role.middleware.js
│ │ ├─ jobTitle.middleware.js
│ │ └─ validate.middleware.js
│ │ └─ error.middleware.js
│ │ └─ rateLimit.middleware.js
│ │ └─ helmet.middleware.js
│ │ └─ cors.middleware.js
│ │ └─ bcrypt.middleware.js
│ │ └─ jwt.middleware.js
│ ├─ /services
│ │ ├─ email.service.js
│ │ ├─ notification.service.js
│ │ └─ scheduler.service.js
│ ├─ /jobs
│ │ ├─ reminder.job.js
│ │ └─ dueDateChecker.job.js
│ ├─ utils/
│ │ ├─ jwt.js
│ │ └─ apiResponse.js
│ ├─ app.js
├─ tests/
│ ├─ auth.test.js
│ ├─ tasks.test.js
│ └─ projects.test.js
└─ package.json
└─ .gitignor

---

# Required Libraries:

    - express — HTTP server framework.

    - mongoose — ODM for working with MongoDB.

    - bcryptjs or bcrypt — Password hashing.

    - jsonwebtoken — JWT generation and verification for authentication.

    - joi or Zod or express-validator — Input validation.

    - helmet — HTTP header security.

    - helmet-csp (optional) — CSP (Content Security Policy) configuration.

    - cors — Cross-origin access control.

    - express-rate-limit — Protection against brute-force attacks.

    - morgan — HTTP request logging during development.

    - winston or pino — Production-grade logging.

    - dotenv — Loading environment variables.

    - nodemailer — Sending emails (invitations/notifications).

    - socket.io (optional) — Real-time updates and notifications.

    - bull, agenda, or bee-queue — Background jobs and scheduling periodic tasks (Redis-backed tools like Bull are preferred).

    - node-cron , Agenda , Bull — For scheduled tasks.

    - multer — File uploads (attachments).

    - swagger-jsdoc + swagger-ui-express — API documentation.

    - chai / mocha / jest / supertest — Testing frameworks.

---

# Database Models:

## User:

    - _id : ObjectId
    - name : [String , required]
    - email : [String ,required , unique , ]
    - password : String (Hashed)
    - avatarUrl : String
    - role : enum [user , admin]
    - createdAt, updatedAt : Date => timestamp

## Project

- \_id : ObjectId
- name : string , required
- description : string
- duration : Number , required (Number of hours)
- cost : Number , required
- owner : objectId (reference for User model)
- createdAt, updatedAt : Date => timestamp

## ProjectMember

- projectId : objectId (reference for Project model)
- userId : objectId (reference for User model)
- jobTitle : enum [manager , teamleader , member] , required
- invitStatus : enum [Pending (defult value) , Accepted , Rejected] , required
- createdAt, updatedAt : Date => timestamp

## Task

- \_id : ObjectId
- projectId : objectId (reference for Project model) , required
- createdBy : objectId (reference for User model) , required
- assignedTo : objectId (reference for User model) , required
- title : String, required
- description : string , required
- attachment : string (like pdf , doc, ....)
- dueDate: date , required
- priority : required [less (defult value) - medium - high]
- status : required , enum [pending (defult value) - inProgress - completed - cancelled]
- createdAt, updatedAt : Date => timestamp

## LogTask

    - _id : objectId
    - taskId : required , objectId (reference for Task model)
    - status : required , enum [pending (defult value) - inProgress - completed - cancelled]
    - description : string
    - createdAt, updatedAt : Date => timestamp

## Notification

    - _id: objectId
    - userId: ObjectId (reference for Task model) , required,
    - type: String, enum: [taskAssigned, taskDue, projectInvite], required,
    - referenceId: ObjectId ( task id OR project id => Polymorphic Relations)
    - message: string (message notification)
    - readable: Boolean (default: false)

---

# 📌 API Endpoints:

## Auth

### POST /api/auth/register

- Description: Create a new user account

- Authentication: Not required

- Request Body: name , email , password , avatar.

- Success Response (201):

  json :
  {
  "success": true,
  "message": "User registered successfully",
  "data": {
  "user": { ... },
  "token": "..."
  }
  }

- Error Responses:

  400: Email already exists

  400: Password too short

  400: Missing required fields

### POST /api/auth/login

- Description: User login

- Authentication: Not required

- Request Body:

  email (String, required)

  password (String, required)

- Success Response (200):
  json Responses:
  {
  "success": true,
  "token": "..."
  }

- Error Responses:

  401: Invalid credentials

### POST /api/auth/refresh

- Description: Refresh access token

- Authentication: Required (Refresh Token)

## Users

### GET /api/users/me

- Description: Retrieve current user profile

- Authentication: Required

- Success Response (200): User object

### PUT /api/users/me

- Description: Update user profile (avatar, name, etc.)

- Authentication: Required

## Projects

## POST /api/projects

- Description: Create a new project

- Authentication: Required

- Request Body:

  name (String, required)

  description (String, optional)

  duration (Number, required)

  cost (Number, required)

- Success Response (201): Project object

## PUT /api/projects

- Description: Update project

- Authentication: Required

- Request Body:

  name (String)

  description (String)

  duration (Number)

  cost (Number)

- Success Response (200): updated Project object

## GET /api/projects

- Description: Retrieve all projects owned or joined by the user

- Authentication: Required

## GET /api/projects/:id

- Description: Retrieve details of a specific project

- Authentication: Required

## DELETE /api/projects/:id

- Description: Delete a project (owner only)

- Authentication: Required

## Project Members

### POST /api/projects/:id/members

- Description: add a new member to the project

- Authentication: Required (Owner/Manager)

- Request Body:

  userId (ObjectId, required)

  jobTitle (Enum: manager, teamLeader, member)

- Success Response (201):
  Invitation object

### PUT /api/projects/:id/members/:memberId

- Description: Update invitation status (Accepted/Rejected)

- Authentication: Required

### DELETE /api/projects/:id/members/:memberId

- Description: Remove a member from the project

- Authentication: Required (Owner/Manager)

## Tasks

### POST /api/tasks

- Description: Create a new task (personal or project-based)

- Authentication: Required

- Request Body:

  title (String, required)

  description (String, required)

  projectId (ObjectId, optional)

  assignedTo (ObjectId, required)

  dueDate (Date, required)

  priority (Enum: low, medium, high)

- Success Response (201): Task object

### GET /api/tasks

- Description: Retrieve all tasks with search and filter options ()

- Authentication: Required

- Query Parameters:

  status (Enum: pending, inProgress, completed)

  projectId (ObjectId)

  assignedTo (ObjectId)

  search (String)

  dueFrom (Date)

  dueTo (Date)

- Success Response (200): List of tasks

### GET /api/tasks/:id

- Description: Retrieve details of a specific task

- Authentication: Required

### PATCH /api/tasks/:id

- Description: Update task details (description, priority, dueDate)

- Authentication: Required

### PATCH /api/tasks/:id/status

- Description: Update task status (pending → inProgress → completed)

- Authentication: Required (assignee or project manager)

- Success Response (200): Updated status + LogTask entry

### POST /api/tasks/:id/assign

- Description: Assign a task to a project member

- Authentication: Required (Owner/Manager)

## LogTasks

### GET /api/tasks/:id/logs

- Description: Retrieve task status change history

- Authentication: Required

## Notifications

### GET /api/notifications

- Description: Retrieve user notifications

- Authentication: Required

### PATCH /api/notifications/:id/read

- Description: Mark notification as read

- Authentication: Required

## Analytics

### GET /api/analytics/tasks/completion

- Description: Task completion rate by project or user

- Authentication: Required

### GET /api/analytics/projects/:id/overview

- Description: Project statistics (task count, completion rate, overdue tasks)

- Authentication: Required

---

# Middleware:

    ## auth.middleware:
        Verify the JWT in the Authorization: Bearer <token> header and attach the user to the request.

    ## role.middleware:
        Checks the user's role within the application (user - admin - ...).

    ## jobTitle.middleware:
        Checks the user's role within the project (manager / teamleader / member) based on the projectId and the current user ID.

    ## validate.middleware:
        Apply express-validator rules and return structured validation errors.

    ## error.middleware:
        A centralized error handler that restricts error messages and unifies the response format.

    ## rateLimit.middleware:
        Apply rate limiting to authentication and invitation routes.

    ## helmet + cors:
        Security headers and access control policies.

    ## bcrypt:
        Store only the passwordHash and validate the password during login.

    ## JWT flow:
        On registration/login, generate a token with a payload containing { userId } and an expiration time (e.g., 7 days). Secret keys are loaded from .env.

---

# 🔐 Authentication

## Overview

- The system uses JWT (JSON Web Tokens) for authentication and bcrypt for secure password hashing.

- Access Tokens: Short‑lived tokens used to access protected routes.

- Refresh Tokens: Long‑lived tokens used to renew access tokens securely.

- Secret Keys: Loaded from .env file (JWT_SECRET, JWT_REFRESH_SECRET).

## Flow

### User Registration (POST /api/auth/register)

- Password is hashed using bcrypt before saving to the database.

- A JWT access token is generated with payload { userId }.

- A refresh token is also generated and stored securely.

### User Login (POST /api/auth/login)

- Credentials are validated.

- If valid, issue a new access token + refresh token.

- If invalid, return 401 Unauthorized.

### Accessing Protected Routes

Client must include the access token in the Authorization header:

- Authorization: Bearer <access_token>
- auth.middleware.js verifies the token and attaches the user object to req.user.
- If token is missing or invalid → 401 Unauthorized.

### Token Refresh (POST /api/auth/refresh)

- Client sends refresh token.

- If valid, issue a new access token.

- If invalid or expired → 403 Forbidden.

### Logout (optional)

Invalidate refresh token (remove from DB or blacklist).

# Error Handling

## Error Categories

### 400 Bad Request

- Cause: Invalid or missing request data

- Examples:

  Missing required fields in registration

  Invalid email format

  Invalid query parameters (e.g., wrong status filter)

### 401 Unauthorized

- Cause: Missing or invalid authentication token

- Examples:

  No Authorization header provided

  Expired or malformed JWT

### 403 Forbidden

- Cause: User is authenticated but lacks permission

- Examples:

  Member trying to delete a project

  User without manager role assigning tasks

### 404 Not Found

- Cause: Requested resource does not exist

- Examples:

  Project ID not found

  Task ID not found

### 409 Conflict

- Cause: Resource conflict

- Examples:

  Registering with an existing email

  Inviting a user already in the project

### 429 Too Many Requests

- Cause: Rate limit exceeded

- Examples:

Too many login attempts (brute force protection)

### 500 Internal Server Error

- Cause: Unexpected server error

- Examples:

  Database connection failure

  Unhandled exceptions in controllers

## Best Practices Implemented

- Validation errors are caught early by validate.middleware.js.

- Authentication errors handled by auth.middleware.js.

- Authorization errors handled by role.middleware.js and jobTitle.middleware.js.

- Centralized error handler ensures all errors return consistent JSON.

- Logging: Detailed error logs are written using winston or pino for debugging, while client receives safe messages.

- Correlation IDs: Each error response can include a unique request ID for tracing in logs.
