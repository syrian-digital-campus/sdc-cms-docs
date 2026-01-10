# Complete API Documentation

This document provides complete API reference for SDC-CMS backend.

## Base URL

```
http://localhost:8080/api/v1
```

## Authentication

Most endpoints require authentication via JWT token.

### Login

**POST** `/auth/login`

Authenticate user and receive access token.

**Request Body**:
```json
{
  "username": "student1",
  "password": "password123"
}
```

**Response** (200 OK):
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "userId": "123e4567-e89b-12d3-a456-426614174000",
  "username": "student1",
  "email": "student1@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "globalRole": "STUDENT"
}
```

**Response Headers**:
- `Set-Cookie: refreshToken=...; HttpOnly; Secure; SameSite=Strict`

### Register

**POST** `/auth/register`

Register new user (if registration is enabled).

**Request Body**:
```json
{
  "username": "newuser",
  "email": "newuser@example.com",
  "password": "securepassword123",
  "firstName": "Jane",
  "lastName": "Smith",
  "globalRole": "STUDENT"
}
```

**Response** (201 Created):
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "userId": "...",
  "username": "newuser",
  // ... user info
}
```

### Refresh Token

**POST** `/auth/refresh`

Get new access token using refresh token.

**Request**: Cookie with `refreshToken`

**Response** (200 OK):
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

### Get Current User

**GET** `/auth/me`

Get current authenticated user information.

**Headers**: `Authorization: Bearer <accessToken>`

**Response** (200 OK):
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "username": "student1",
  "email": "student1@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "globalRole": "STUDENT",
  "matriculationNumber": "12345",
  "officeLocation": null,
  "languagePreference": "en"
}
```

### Logout

**POST** `/auth/logout`

Logout user (clears refresh token cookie).

**Headers**: `Authorization: Bearer <accessToken>`

**Response** (200 OK): Empty body

## Courses

### Get All Courses

**GET** `/courses`

Get list of all courses (public, but shows enrollment status if authenticated).

**Headers**: `Authorization: Bearer <accessToken>` (optional)

**Response** (200 OK):
```json
[
  {
    "id": "...",
    "title": "Formal Analysis of Real-World Security Protocols",
    "description": "...",
    "courseCode": "FARWSP",
    "semester": "WINTER",
    "year": 2025,
    "ownerId": "...",
    "ownerName": "Prof. Cremers",
    "createdAt": "2024-12-01T10:00:00Z",
    "isEnrolled": true,
    "enrollmentRole": {
      "role": "STUDENT",
      "userId": "...",
      "firstName": "John",
      "lastName": "Doe"
    },
    "slug": "farwsp25"
  }
]
```

### Get Course by Slug

**GET** `/courses/slug/{slug}`

Get course by slug (human-readable identifier).

**Path Parameters**:
- `slug`: Course slug (e.g., "farwsp25")

**Response**: Same as single course

### Get Course by ID

**GET** `/courses/{courseId}`

Get course by UUID.

**Response** (200 OK): Course object

### Create Course

**POST** `/courses`

Create new course (requires ADMIN or PROFESSOR role).

**Headers**: `Authorization: Bearer <accessToken>`

**Request Body**:
```json
{
  "title": "New Course",
  "description": "Course description",
  "courseCode": "NEWC",
  "semester": "WINTER",
  "year": 2025
}
```

**Response** (201 Created): Course object

### Update Course

**PUT** `/courses/{courseId}`

Update course (requires course owner or ADMIN).

**Request Body**: Same as create (all fields optional)

**Response** (200 OK): Updated course

### Delete Course

**DELETE** `/courses/{courseId}`

Delete course (requires course owner or ADMIN).

**Response** (204 No Content)

## Enrollments

### Enroll in Course

**POST** `/courses/{courseId}/enroll`

Enroll in course (requires authentication, registration must be enabled).

**Request Body** (optional):
```json
{
  "role": "STUDENT"  // Optional, defaults to STUDENT
}
```

**Response** (201 Created): Enrollment object

### Unenroll from Course

**DELETE** `/courses/{courseId}/enroll`

Unenroll from course.

**Response** (204 No Content)

### Get Course Participants

**GET** `/courses/{courseId}/participants`

Get all participants in a course (enrolled users).

**Response** (200 OK):
```json
[
  {
    "role": "PROFESSOR",
    "userId": "...",
    "firstName": "Prof.",
    "lastName": "Cremers",
    "email": "cremers@example.com"
  },
  {
    "role": "STUDENT",
    "userId": "...",
    "firstName": "John",
    "lastName": "Doe",
    "email": "student1@example.com"
  }
]
```

## Materials

### Get Materials

**GET** `/courses/{courseId}/materials`

Get materials for a course.

**Query Parameters**:
- `includeHidden` (boolean, default: false): Include hidden materials (staff only)

**Response** (200 OK):
```json
[
  {
    "id": "...",
    "courseId": "...",
    "title": "Lecture 1 Slides",
    "description": "Introduction slides",
    "fileInfo": {
      "id": "...",
      "originalFilename": "lecture1.pdf",
      "contentType": "application/pdf",
      "fileSize": 1024000
    },
    "visibleFrom": "2024-12-01T00:00:00Z",
    "visibleUntil": null,
    "displayOrder": 1,
    "isVisible": true,
    "createdByName": "Prof. Cremers",
    "createdAt": "2024-12-01T10:00:00Z"
  }
]
```

### Create Material

**POST** `/courses/{courseId}/materials`

Create material (requires staff role).

**Content-Type**: `multipart/form-data`

**Form Data**:
- `title` (required): Material title
- `description` (optional): Description
- `file` (optional): File to upload
- `visibleFrom` (optional): ISO 8601 date string
- `visibleUntil` (optional): ISO 8601 date string
- `displayOrder` (optional): Integer

**Response** (201 Created): Material object

### Get Material

**GET** `/courses/{courseId}/materials/{materialId}`

Get single material.

**Response** (200 OK): Material object

### Update Material

**PUT** `/courses/{courseId}/materials/{materialId}`

Update material (requires staff role).

**Content-Type**: `multipart/form-data`

**Form Data**: Same as create (all fields optional)

**Response** (200 OK): Updated material

### Delete Material

**DELETE** `/courses/{courseId}/materials/{materialId}`

Delete material (requires staff role).

**Response** (204 No Content)

### Download Material File

**GET** `/courses/{courseId}/materials/{materialId}/download`

Download material file.

**Response** (200 OK):
- Content-Type: File MIME type
- Content-Disposition: attachment; filename="..."
- Body: File binary data

## Assignments

### Get Assignments

**GET** `/courses/{courseId}/assignments`

Get assignments for a course.

**Query Parameters**:
- `includeHidden` (boolean, default: false): Include hidden assignments (staff only)

**Response** (200 OK):
```json
[
  {
    "id": "...",
    "courseId": "...",
    "title": "Assignment 1",
    "description": "Complete the exercises",
    "dueDate": "2024-12-31T23:59:59Z",
    "maxPoints": 100.0,
    "visibleFrom": "2024-12-01T00:00:00Z",
    "specFileInfo": {
      "id": "...",
      "originalFilename": "assignment1.pdf",
      "contentType": "application/pdf",
      "fileSize": 512000
    },
    "isVisible": true,
    "isOverdue": false,
    "createdByName": "Prof. Cremers",
    "createdAt": "2024-12-01T10:00:00Z"
  }
]
```

### Create Assignment

**POST** `/courses/{courseId}/assignments`

Create assignment (requires staff role).

**Content-Type**: `multipart/form-data`

**Form Data**:
- `title` (required): Assignment title
- `description` (optional): Description
- `dueDate` (required): ISO 8601 date string
- `maxPoints` (required): Decimal number
- `visibleFrom` (optional): ISO 8601 date string
- `specFile` (optional): Specification file

**Response** (201 Created): Assignment object

### Get Assignment

**GET** `/courses/{courseId}/assignments/{assignmentId}`

Get single assignment.

**Response** (200 OK): Assignment object

### Update Assignment

**PUT** `/courses/{courseId}/assignments/{assignmentId}`

Update assignment (requires staff role).

**Content-Type**: `multipart/form-data`

**Form Data**: Same as create (all fields optional)

**Response** (200 OK): Updated assignment

### Delete Assignment

**DELETE** `/courses/{courseId}/assignments/{assignmentId}`

Delete assignment (requires staff role).

**Response** (204 No Content)

### Download Assignment Spec

**GET** `/courses/{courseId}/assignments/{assignmentId}/spec/download`

Download assignment specification file.

**Response** (200 OK): File binary data

## Submissions

### Create Submission

**POST** `/courses/{courseId}/assignments/{assignmentId}/submissions`

Create submission for assignment.

**Content-Type**: `multipart/form-data`

**Form Data**:
- `files` (required): Array of files
- `note` (optional): Submission note

**Response** (201 Created):
```json
{
  "id": "...",
  "assignmentId": "...",
  "userId": "...",
  "username": "student1",
  "userFullName": "John Doe",
  "submittedAt": "2024-12-15T10:00:00Z",
  "status": "SUBMITTED",
  "grade": null,
  "feedbackText": null,
  "files": [
    {
      "id": "...",
      "fileInfo": {
        "id": "...",
        "originalFilename": "solution.pdf",
        "contentType": "application/pdf",
        "fileSize": 256000
      },
      "uploadedAt": "2024-12-15T10:00:00Z"
    }
  ],
  "isLate": false,
  "createdAt": "2024-12-15T10:00:00Z"
}
```

### Get Submissions

**GET** `/courses/{courseId}/assignments/{assignmentId}/submissions`

Get submissions for assignment.
- Students see only their own submission
- Staff see all submissions

**Response** (200 OK): Array of submission objects

### Get Submission

**GET** `/courses/{courseId}/assignments/{assignmentId}/submissions/{submissionId}`

Get single submission.

**Response** (200 OK): Submission object

### Grade Submission

**POST** `/courses/{courseId}/assignments/{assignmentId}/submissions/{submissionId}/grade`

Grade submission (requires staff role).

**Content-Type**: `multipart/form-data`

**Form Data**:
- `grade` (required): Decimal number
- `feedbackText` (optional): Text feedback
- `correctionFile` (optional): Correction file

**Response** (200 OK): Updated submission

## Error Responses

All endpoints may return error responses:

**400 Bad Request**:
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Title is required"
}
```

**401 Unauthorized**:
```json
{
  "error": "UNAUTHORIZED",
  "message": "Authentication required"
}
```

**403 Forbidden**:
```json
{
  "error": "FORBIDDEN",
  "message": "You don't have permission to perform this action"
}
```

**404 Not Found**:
```json
{
  "error": "NOT_FOUND",
  "message": "Course not found: 123e4567..."
}
```

**500 Internal Server Error**:
```json
{
  "error": "INTERNAL_ERROR",
  "message": "An unexpected error occurred"
}
```

## Rate Limiting

Currently not implemented, but recommended for production:
- Limit: X requests per minute per IP/user
- Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`

## Pagination

Currently not implemented. All list endpoints return all results.

Future implementation:
- Query parameters: `page`, `size`
- Response headers: `X-Total-Count`, `Link` (for pagination links)

## Next Steps

- Test endpoints using Postman or curl
- See [07_DEVELOPMENT_GUIDE.md](./07_DEVELOPMENT_GUIDE.md) for development workflows
- See backend `TESTING_GUIDE.md` for endpoint testing examples

---

**Document Version**: 1.0  
**Last Updated**: December 2024




