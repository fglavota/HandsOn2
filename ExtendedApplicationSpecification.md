# Code Connect - Extended Application Specification

## Overview

**Mission:**  
Build "Code Connect," a social platform for developers to share code snippets, discuss programming challenges, collaborate on projects, and incorporate additional functionalities to enhance collaboration, engagement, and discovery.

**Key Features (Including Extensions):**  
- **User Profiles:** Basic info, bio, skills, profile pictures, and GitHub/GitLab links.
- **Code Snippets:** Post/share code with syntax highlighting, titles, descriptions, tags, and the ability to fork.
- **Comments & Discussions:** Users can comment on snippets and discuss coding solutions.
- **Project Collaboration:** Create/join projects, share repos, assign tasks, and track progress.
- **Task Management:** Within projects, create/assign tasks, set priorities and due dates.
- **User Interaction:** Follow other users, favorite (star) snippets, and receive notifications.
- **Search & Discover:** Find snippets, projects, users by keywords, tags, language, and filter/sort results.
- **Notifications:** Notify users about new comments, follows, task assignments, etc.

---

## Data Models

### 1. User Model
**Fields:**
- `id` (string | UUID)
- `username` (string, required, unique)
- `name` (string, optional)
- `bio` (string, optional)
- `skills` (array of strings, optional)
- `githubProfileUrl` (string, optional)
- `gitlabProfileUrl` (string, optional)
- `profileImageUrl` (string, optional)
- `createdAt` (datetime)
- `updatedAt` (datetime)

**Indices:**
- Primary: `id`
- Unique: `username`

**Relationships:**
- A User can have many Snippets.
- A User can have many Comments.
- A User can be a member of many Projects.
- A User can follow many Users and be followed by many Users.
- A User can have many Favorites (Snippets they favorited).
- A User can have many Notifications.

### 2. Snippet Model
**Fields:**
- `id` (string | UUID)
- `ownerId` (string | UUID, required)
- `title` (string, required)
- `description` (string, optional)
- `code` (string, required)
- `language` (string, required)
- `tags` (array of strings, optional)
- `forkedFromId` (string | UUID, optional) – ID of snippet forked from
- `createdAt` (datetime)
- `updatedAt` (datetime)

**Indices:**
- Primary: `id`
- Index: `ownerId`, `tags`, `language`

**Relationships:**
- A Snippet belongs to a User.
- A Snippet has many Comments.
- A Snippet can be favorited by many Users.
- A Snippet can be forked from another Snippet.

### 3. Comment Model
**Fields:**
- `id` (string | UUID)
- `snippetId` (string | UUID, required)
- `authorId` (string | UUID, required)
- `content` (string, required)
- `createdAt` (datetime)
- `updatedAt` (datetime)

**Indices:**
- Primary: `id`
- Index: `snippetId`, `authorId`

**Relationships:**
- A Comment belongs to a Snippet.
- A Comment is authored by a User.

### 4. Project Model
**Fields:**
- `id` (string | UUID)
- `ownerId` (string | UUID, required)
- `name` (string, required)
- `description` (string, optional)
- `repoUrl` (string, optional)
- `createdAt` (datetime)
- `updatedAt` (datetime)

**Indices:**
- Primary: `id`
- Index: `ownerId`

**Relationships:**
- A Project belongs to a User (owner).
- A Project has multiple members (Users) via ProjectMembership.
- A Project has many Tasks.

### 5. ProjectMembership Model
**Fields:**
- `id` (string | UUID)
- `projectId` (string | UUID, required)
- `userId` (string | UUID, required)
- `role` (string, optional): e.g. `"maintainer"`, `"contributor"`
- `createdAt` (datetime)
- `updatedAt` (datetime)

**Indices:**
- Primary: `id`
- Unique compound: `(projectId, userId)`

**Relationships:**
- Links a User to a Project.

### 6. Task Model
**Fields:**
- `id` (string | UUID)
- `projectId` (string | UUID, required)
- `title` (string, required)
- `description` (string, optional)
- `assignedToUserId` (string | UUID, optional)
- `status` (string, required): `"open"`, `"in-progress"`, `"review"`, `"closed"`
- `priority` (string, optional): `"low"`, `"medium"`, `"high"`
- `dueDate` (datetime, optional)
- `createdAt` (datetime)
- `updatedAt` (datetime)

**Indices:**
- Primary: `id`
- Index: `projectId`, `assignedToUserId`

**Relationships:**
- A Task belongs to a Project.
- A Task may be assigned to a User.

### 7. Follow Model
**Fields:**
- `id` (string | UUID)
- `followerId` (string | UUID, required)
- `followedId` (string | UUID, required)
- `createdAt` (datetime)

**Indices:**
- Primary: `id`
- Unique compound: `(followerId, followedId)`

**Relationships:**
- A Follow links one User (follower) to another User (followed).

### 8. Favorite Model
**Fields:**
- `id` (string | UUID)
- `userId` (string | UUID, required)
- `snippetId` (string | UUID, required)
- `createdAt` (datetime)

**Indices:**
- Primary: `id`
- Unique compound: `(userId, snippetId)`

**Relationships:**
- A Favorite links a User to a Snippet.

### 9. Notification Model
**Fields:**
- `id` (string | UUID)
- `userId` (string | UUID, required)
- `type` (string, required): `"comment"`, `"follow"`, `"task_assigned"`, etc.
- `referenceId` (string | UUID, optional)
- `message` (string, required)
- `isRead` (boolean, default: false)
- `createdAt` (datetime)
- `updatedAt` (datetime)

**Indices:**
- Primary: `id`
- Index: `userId`, `isRead`

**Relationships:**
- A Notification belongs to a User.

---

## API Endpoints

**Common Responses:**
- `200 OK` for success
- `201 Created` for resource creation
- `204 No Content` for successful deletion
- `400 Bad Request` for invalid input
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- `500 Internal Server Error`

### Users & Follows

**POST /users**  
- Create a user.

**GET /users/{userId}**  
- Retrieve user profile.

**PUT /users/{userId}**  
- Update user profile.

**POST /users/{userId}/follow**  
- Follow another user.

**DELETE /users/{userId}/follow**  
- Unfollow a user.

**GET /users/{userId}/followers**  
- List followers of a user.

**GET /users/{userId}/following**  
- List users that the user is following.

### Snippets & Favorites

**POST /snippets**  
- Create a snippet.

**GET /snippets/{snippetId}**  
- Get a snippet by ID.

**PUT /snippets/{snippetId}**  
- Update snippet.

**DELETE /snippets/{snippetId}**  
- Delete snippet.

**POST /snippets/{snippetId}/favorite**  
- Favorite a snippet.

**DELETE /snippets/{snippetId}/favorite**  
- Unfavorite a snippet.

**GET /users/{userId}/favorites**  
- List a user's favorited snippets.

**POST /snippets/{snippetId}/fork**  
- Fork a snippet.

### Comments

**POST /comments/{snippetId}**  
- Add a comment to a snippet.

**GET /comments/{snippetId}**  
- Retrieve comments for a snippet.

### Projects & Membership

**POST /projects**  
- Create a project.

**GET /projects/{projectId}**  
- Get project details.

**PUT /projects/{projectId}**  
- Update project info.

**POST /projects/{projectId}/members**  
- Add a user to a project.

### Tasks

**POST /projects/{projectId}/tasks**  
- Create a task in a project.

**GET /projects/{projectId}/tasks**  
- List tasks in a project (filter by status, assigned user, priority).

**GET /tasks/{taskId}**  
- Get a specific task.

**PUT /tasks/{taskId}**  
- Update a task (change status, reassign, etc.).

**DELETE /tasks/{taskId}**  
- Delete a task.

### Notifications

**GET /notifications**  
- Get notifications for the authenticated user.

**PUT /notifications/{notificationId}/read**  
- Mark a notification as read.

### Search & Discover

**GET /search/snippets**  
- Params: `q`, `tags`, `language`, `sort`, `page`, `limit`  
- Returns filtered, sorted, paginated snippets.

**GET /search/users**  
- Params: `q`, `skills`, `page`, `limit`  
- Returns filtered, paginated users.

**GET /search/projects**  
- Params: `q`, `page`, `limit`  
- Returns filtered, paginated projects.

---

## Unit Test Specifications

### Users & Follows
- Creating a user with valid data returns `201`.
- Retrieving a user by ID returns `200` if found, `404` if not.
- Updating a user with valid token and data returns `200`.
- Following a user returns `201`, unfollowing returns `204`.
- Listing followers/following returns `200` and correct lists.

### Snippets & Favorites
- Creating a snippet returns `201`.
- Getting a snippet by valid ID returns `200`, invalid returns `404`.
- Updating snippet by owner returns `200`, unauthorized returns `403`.
- Deleting a snippet by owner returns `204`.
- Favoriting a snippet returns `201`, unfavoriting returns `204`.
- Listing favorites returns `200` with correct snippets.
- Forking a snippet returns `201` and sets `forkedFromId`.

### Comments
- Adding a comment returns `201`.
- Retrieving comments returns `200` and correct list.
- Comments on non-existent snippet returns `404`.

### Projects & Membership
- Creating a project returns `201`.
- Getting project returns `200`, non-existent `404`.
- Updating project by owner returns `200`, unauthorized `403`.
- Adding member to project returns `201`.
- Non-owner attempt to add member returns `403`.

### Tasks
- Creating a task returns `201`.
- Listing tasks returns `200` and filtered results.
- Getting a task returns `200` if found.
- Updating task (e.g., status) returns `200`.
- Deleting task returns `204`.

### Notifications
- Retrieving notifications returns `200`.
- Marking notification as read returns `200`.
- Non-existent notification returns `404`.

### Search
- Searching snippets/users/projects with valid queries returns `200`.
- Invalid query parameters return `400`.
- Pagination and sorting produce correct, testable results.

---

By integrating these models, endpoints, and unit tests, Code Connect gains richer functionality—enhanced project collaboration with tasks, user engagement through follows and favorites, snippet forking for iterative code improvement, notifications for timely updates, and robust search capabilities. This specification can guide both the development process and provide structure for automated testing to ensure reliability and quality.
