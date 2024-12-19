# Comprehensive API Models and Specifications

## Data Models

### 1. User Model
**Description:** Represents a registered user of the platform.

**Suggested Fields:**
- `id` (string | UUID): Unique identifier for the user.
- `username` (string, required, unique): Chosen username.
- `name` (string, optional): The user’s real name.
- `bio` (string, optional): A short bio describing the user.
- `skills` (array of strings, optional): List of programming skills, e.g., `["JavaScript", "Python"]`.
- `githubProfileUrl` (string, optional): Link to GitHub profile.
- `gitlabProfileUrl` (string, optional): Link to GitLab profile.
- `profileImageUrl` (string, optional): URL to the user’s profile picture.
- `createdAt` (datetime): Timestamp when the user was created.
- `updatedAt` (datetime): Timestamp when the user was last updated.

**Indices:**
- Primary: `id`
- Unique: `username`

**Relationships:**
- A User can have many Snippets.
- A User can have many Comments.
- A User can participate in many Projects (through a relationship table if needed).

---

### 2. Snippet Model
**Description:** Represents a code snippet shared by a user.

**Suggested Fields:**
- `id` (string | UUID): Unique identifier for the snippet.
- `ownerId` (string | UUID, required): The ID of the user who created the snippet.
- `title` (string, required): A short title for the snippet.
- `description` (string, optional): Detailed description of what the snippet does.
- `code` (string, required): The actual code content.
- `language` (string, required): Programming language (e.g., `"JavaScript"`, `"Python"`, `"Go"`).
- `tags` (array of strings, optional): Tags for categorization (e.g., `["frontend", "authentication"]`).
- `createdAt` (datetime): Timestamp of creation.
- `updatedAt` (datetime): Timestamp of last update.

**Indices:**
- Primary: `id`
- Index: `ownerId`, `tags`, `language`

**Relationships:**
- A Snippet belongs to a User (owner).
- A Snippet has many Comments.

---

### 3. Comment Model
**Description:** Represents a user’s comment on a snippet.

**Suggested Fields:**
- `id` (string | UUID): Unique identifier for the comment.
- `snippetId` (string | UUID, required): The ID of the snippet the comment is on.
- `authorId` (string | UUID, required): The ID of the user who posted the comment.
- `content` (string, required): The comment text.
- `createdAt` (datetime): Timestamp of creation.
- `updatedAt` (datetime): Timestamp of last update.

**Indices:**
- Primary: `id`
- Index: `snippetId`, `authorId`

**Relationships:**
- A Comment belongs to a Snippet.
- A Comment is authored by a User.

---

### 4. Project Model
**Description:** Represents a collaborative project that users can join and share code repositories.

**Suggested Fields:**
- `id` (string | UUID): Unique identifier for the project.
- `ownerId` (string | UUID, required): The ID of the user who created the project.
- `name` (string, required): Project name.
- `description` (string, optional): Brief description of the project.
- `repoUrl` (string, optional): URL to the main code repository (e.g., GitHub link).
- `createdAt` (datetime): Timestamp of creation.
- `updatedAt` (datetime): Timestamp of last update.

**Indices:**
- Primary: `id`
- Index: `ownerId`

**Relationships:**
- A Project belongs to a User (owner).
- A Project has many members (Users) who collaborate on it.
  - This may be implemented via a `ProjectMembership` join table with fields: `id`, `projectId`, `userId`, `role`.

---

### 5. ProjectMembership Model (If needed)
**Description:** Connects users and projects, indicating who is a member of which project.

**Suggested Fields:**
- `id` (string | UUID): Unique identifier.
- `projectId` (string | UUID, required): The ID of the project.
- `userId` (string | UUID, required): The ID of the user who is a member.
- `role` (string, optional): Role in the project, e.g., `["maintainer", "contributor"]`.
- `createdAt` (datetime)
- `updatedAt` (datetime)

**Indices:**
- Primary: `id`
- Unique compound index: (`projectId`, `userId`)

---

## API Endpoints

### Base URL
`https://api.codeconnect.com/v1`

**Standard HTTP Status Codes:**
- `200 OK` for successful GET/PUT/POST requests.
- `201 Created` for successful resource creation.
- `204 No Content` for successful deletions.
- `400 Bad Request` for invalid input.
- `401 Unauthorized` if the user is not authenticated.
- `403 Forbidden` if the user is not authorized to access the resource.
- `404 Not Found` if the requested resource is not found.
- `500 Internal Server Error` for unexpected errors.

### Endpoints

#### `/users`

**POST** `/users`

**Description:** Create a new user account.

**Request Body (JSON):**
```json
{
  "username": "john_doe",
  "name": "John Doe",
  "bio": "Full-stack developer.",
  "skills": ["JavaScript", "React"],
  "githubProfileUrl": "https://github.com/john_doe",
  "gitlabProfileUrl": "https://gitlab.com/john_doe",
  "profileImageUrl": "https://example.com/images/john.jpg"
}
```

**Response (201 Created):**
```json
{
  "id": "user-uuid",
  "username": "john_doe",
  "name": "John Doe",
  "bio": "Full-stack developer.",
  "skills": ["JavaScript", "React"],
  "githubProfileUrl": "https://github.com/john_doe",
  "gitlabProfileUrl": "https://gitlab.com/john_doe",
  "profileImageUrl": "https://example.com/images/john.jpg",
  "createdAt": "2024-12-17T10:00:00Z",
  "updatedAt": "2024-12-17T10:00:00Z"
}
```

**GET** `/users/{userId}`

**Description:** Retrieve user profile details.

**Response (200 OK):**
```json
{
  "id": "user-uuid",
  "username": "john_doe",
  "name": "John Doe",
  "bio": "Full-stack developer.",
  "skills": ["JavaScript", "React"],
  "githubProfileUrl": "https://github.com/john_doe",
  "gitlabProfileUrl": "https://gitlab.com/john_doe",
  "profileImageUrl": "https://example.com/images/john.jpg",
  "createdAt": "2024-12-17T10:00:00Z",
  "updatedAt": "2024-12-17T10:00:00Z"
}
```

...

*(The rest of the endpoints follow the same detailed pattern.)*

---

## Additional Considerations

### Authentication & Authorization
- Implement authentication endpoints (e.g., `/auth/login`, `/auth/signup`) with token-based security (e.g., JWT).
- Enforce authorization checks to ensure users can only access or modify their resources.

### Pagination, Filtering & Sorting
- Support query parameters like `page`, `limit`, `sortField`, and `sortOrder` for listing endpoints such as `/snippets` or `/comments/{snippetId}`.

### Error Handling
Standardize error responses:
```json
{
  "error": {
    "message": "Invalid user ID",
    "code": 400
  }
}
```

### Validation
Ensure all inputs are validated (e.g., `ownerId` as a valid UUID, `title` and `code` as non-empty strings).

### Versioning
Prefix the API with `/v1` to manage future backward-incompatible changes.
