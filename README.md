# Task Management System API Design

## Project Structure
task-management-system/
├── src/
│ ├── controllers/
│ │ ├── authController.js
│ │ ├── userController.js
│ │ ├── taskController.js
│ │ ├── projectController.js
│ │ └── notificationController.js
│ ├── models/
│ │ ├── User.js
│ │ ├── Task.js
│ │ ├── Project.js
│ │ └── Notification.js
│ ├── routes/
│ │ ├── authRoutes.js
│ │ ├── userRoutes.js
│ │ ├── taskRoutes.js
│ │ ├── projectRoutes.js
│ │ └── notificationRoutes.js
│ ├── middleware/
│ │ ├── authMiddleware.js
│ │ ├── validationMiddleware.js
│ │ └── errorMiddleware.js
│ └── utils/
│   ├── passwordService.js
│   ├── tokenService.js
│   └── cookieService.js
├──app.js
├── package.json
├──.env
├──.gitignore
└── README.md

## Required Libraries
- **express**: Web framework for building api
- **mongoose**: for MongoDB 
- **dotenv**: Load environment variables from .env file
- **morgan**:To show the info of requests
- **nodemon**: Automatically restart server during development i use it,Its better than useing node 
- **bcryptjs**: For password hashing and security
- **jsonwebtoken**: For JWT authentication
- **cookie-parser**: To dealing with cookies


## Database Models

 User Model

{
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  avatar: { type: String, default: '' },
  createdAt: { type: Date, default: Date.now }
}

Project Model
{
  name: { type: String, required: true },
  description: { type: String },
  owner: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  members: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
  createdAt: { type: Date, default: Date.now }
}

Task Model
{
  title: { type: String, required: true },
  description: { type: String },
  status: { 
    type: String, 
    enum: ['todo', 'in progress', 'done'], 
    default: 'todo' 
  },
  priority: { 
    type: String, 
    enum: ['low', 'medium', 'high'], 
    default: 'medium' 
  },
  dueDate: { type: Date },
  assignedTo: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  project: { type: mongoose.Schema.Types.ObjectId, ref: 'Project' },
  createdBy: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
}

Notification Model
{
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  type: { 
    type: String, 
    enum: ['task_assigned', 'deadline_reminder', 'project_invitation'],
    required: true 
  },
  title: { type: String, required: true },
  message: { type: String, required: true },
  relatedTask: { type: mongoose.Schema.Types.ObjectId, ref: 'Task' },
  relatedProject: { type: mongoose.Schema.Types.ObjectId, ref: 'Project' },
  isRead: { type: Boolean, default: false },
  createdAt: { type: Date, default: Date.now }
}

Api :
Authentication Routes

POST /api/auth/register
Method: POST

Description: Create a new user account

Authentication: Not required

Request Body:
( name , email , password )
Response Success (200):
{ success: true, message: "User registered successfully", data: { user: {...} } }

Response Errors:

400: { success: false, error: "Email already exists" }

400: { success: false, error: "Password must be at least 6 characters" }

400: { success: false, error: "All fields are required" }

Potential User Mistakes:

Submitting without filling all required fields

Using an email that already exists

Entering a weak password

POST /api/auth/login

Method: POST

Description: Login user and return JWT token

Authentication: Not required

Request Body:
( name , email , password )
Response Success (200):
{ success: true, message: "Login successful", data: { user: {...} } }

Response Errors:

400: { success: false, error: "All fields are required" }

401: { success: false, error: "Invalid email or password" }

Potential User Mistakes:

Forgetting password

Using wrong email format

Leaving fields empty

POST /api/auth/logout
Method: POST

Description: Logout user and clear tokens

Authentication: Required

Response Success (200):
{ success: true, message: "Logout successful" }

Response Errors:

401: { success: false, error: "Authentication required" }

Potential User Mistakes:

Trying to logout without being logged in

Token already expired

POST api/auth/refreshToken :
Method: POST
Description: Refresh Token for user's account
Authentication: required
Response Success (200): { success: true,{message: "Tokens Refreshed Successfully"}}
Response Errors:
500 : { success: false, error: "serve error" }
Potential User Mistakes:
Submitting without filling all required fields
Using an email is not exists
Entering a wrong password
Internal Server Error

User Routes
GET /api/users/profile
Method: GET

Description: Get current user profile

Authentication: Required 

Response Success (200):
{ success: true, data: { user: {...} } }

Response Errors:

401: { success: false, error: "Authentication required" }

404: { success: false, error: "User not found" }

PUT /api/users/profile
Method: PUT

Description: Update user profile

Authentication: Required 

Request Body:
( name , avatar) 
Response Success (200):
{ success: true, message: "Profile updated successfully", data: { user: {...} } }

Response Errors:

401: { success: false, error: "Authentication required" }

400: { success: false, error: "No fields to update" }

Task Routes
GET /api/tasks
Method: GET

Description: Get all tasks for the logged-in user

Authentication: Required 

Query Parameters:

status (string: 'todo', 'in progress', 'done')

project (string: project ID)

search (string: search in task title)

Response Success (200):
{ success: true, data: [{ task1 }, { task2 }, ...] }

Response Errors:

401: { success: false, error: "Authentication required" }

500: { success: false, error: "Server error" }

Potential User Mistakes:

Using invalid status filter

Searching with empty query

POST /api/tasks
Method: POST

Description: Create a new task

Authentication: Required 

Request Body:
(title ,description ,priority ,dueDate ,project ,assignedTo )
Response Success (201):
{ success: true, message: "Task created successfully", data: { task: {...} } }

Response Errors:

401: { success: false, error: "Authentication required" }

400: { success: false, error: "Title is required" }

404: { success: false, error: "Project not found" }

PUT /api/tasks/:id
Method: PUT

Description: Update a task

Authentication: Required 

Request Body:
(title ,description ,priority ,dueDate ,project ,assignedTo )
Response Success (200):
{ success: true, message: "Task updated successfully", data: { task: {...} } }

Response Errors:

401: { success: false, error: "Authentication required" }

404: { success: false, error: "Task not found" }

403: { success: false, error: "Not authorized to update this task" }

DELETE /api/tasks/:id
Method: DELETE

Description: Delete a task

Authentication: Required 

Response Success (200):
{ success: true, message: "Task deleted successfully" }

Response Errors:

401: { success: false, error: "Authentication required" }

404: { success: false, error: "Task not found" }

403: { success: false, error: "Not authorized to delete this task" }

Project Routes
GET /api/projects
Method: GET

Description: Get all projects for the logged-in user

Authentication: Required 

Response Success (200):
{ success: true, data: [{ project1 }, { project2 }, ...] }

Response Errors:

401: { success: false, error: "Authentication required" }

POST /api/projects
Method: POST

Description: Create a new project

Authentication: Required 

Request Body:
( name , description )
Response Success (201):
{ success: true, message: "Project created successfully", data: { project: {...} } }

Response Errors:

401: { success: false, error: "Authentication required" }

400: { success: false, error: "Project name is required" }

POST /api/projects/:id/members
Method: POST

Description: Add member to project

Authentication: Required 

Request Body:
email 
Response Success (200):
{ success: true, message: "Member added successfully", data: { project: {...} } }

Response Errors:

401: { success: false, error: "Authentication required" }

404: { success: false, error: "Project not found" }

404: { success: false, error: "User not found" }

403: { success: false, error: "Only project owner can add members" }

Notification Routes
GET /api/notifications
Method: GET

Description: Get all notifications for the logged-in user

Authentication: Required 

Query Parameters:

(isRead)

Response Success (200):
{ success: true, data: [{ notification1 }, { notification2 }, ...] }

Response Errors:

401: { success: false, error: "Authentication required" }

PUT /api/notifications/:id/read
Method: PUT

Description: Mark notification as read

Authentication: Required 

Response Success (200):
{ success: true, message: "Notification marked as read" }

Response Errors:

401: { success: false, error: "Authentication required" }

404: { success: false, error: "Notification not found" }

Middleware

Authentication Middleware

authMiddleware: Verify JWT token and attach user to request object

ownerMiddleware: Check if user owns the resource they're trying to modify

Validation Middleware

validateRegister: Validate user registration data

validateLogin: Validate user login data

validateTask: Validate task creation/update data

validateProject: Validate project creation/update data

Error Handling Middleware

errorHandler: Centralized error handling middleware

notFound: Handle 404 errors for undefined routes

Utils Services
Authentication Strategy that i did :
JWT Implementation
Use jsonwebtoken library for token generation and verification

Tokens expire after 7 days for security

Store user ID and email in token payload

Send token in Authorization header as "Bearer {token}"

Cookie Service (src/utils/cookieService.js) , its methods:

setAccessToken: Set JWT Access token in HTTP-only cookie

setRefreshToken: Set JWT Refresh token in HTTP-only cookie

getAccessToken: Extract Access token from cookies for authentication

getRefreshToken: Extract Refresh token from cookies for authentication

clearTokens: Clear authentication cookie on logout

Password Service (src/utils/passwordService.js)
hashPassword(password): Hash plain text password
comparePassword(password, hashedPassword): Compare password with hash

Token Service (src/utils/tokenService.js)

generateAccessToken: Generate JWT Access token
generateRefreshToken: Generate JWT Refresh token

verifyAccessToken: Verify and decode JWT Access token
verifyRefreshToken: Verify and decode JWT Refresh token


----------------------

Error Handling

Authentication Errors: Invalid token, expired token, no token provided

Validation Errors: Missing required fields, invalid data formats

Authorization Errors: User trying to access/modify unauthorized resources

Database Errors: Duplicate entries, connection issues

Not Found Errors: Resources that don't exist

-------------------