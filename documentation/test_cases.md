# Comprehensive Postman Test Cases for Laravel API (Positive & Negative)

This document contains a complete test suite covering all endpoints, including route protection, valid CRUD operations, input validations, error handling, and database integrity checks.

### Environment Setup in Postman
Before running these tests, configure your **Postman Environment** or **Globals**:
* `baseUrl` = `http://127.0.0.1:8000` (or your specific local domain)
* `bearerToken` = `1|ln7v4X...` (the plain-text token printed in your terminal after running `php artisan db:seed`)

> **Critical Header:** Every single request sent to this API must include the following entry under the **Headers** tab:
> * **Key:** `Accept` | **Value:** `application/json`

---

## 1. Authentication & Route Protection

### Test Case 1.1: Request Without Authorization Token (Negative)
* **Objective:** Verify that private resource endpoints reject unauthenticated clients.
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/courses`
* **Authorization:** Select `No Auth` (ensure no `Authorization` header is sent).
* **Expected Result:** 
  * HTTP Status: `401 Unauthorized`
  * Body: `{"message": "Unauthenticated."}`

### Test Case 1.2: Get User Profile with Valid Token (Positive)
* **Objective:** Confirm the Bearer Token is valid and successfully authenticates against the Sanctum guard.
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/user`
* **Authorization:** Select `Bearer Token` -> input `{{bearerToken}}`.
* **Expected Result:** 
  * HTTP Status: `200 OK`
  * Body: JSON object matching the authenticated `Admin User` row (contains `id`, `name`, `email`).

---

## 2. Courses Resource (`/api/courses`)

### Test Case 2.1: Retrieve All Courses (Positive)
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/courses`
* **Authorization:** `Bearer Token`
* **Expected Result:** `200 OK` | A JSON array containing the 20 seeded courses wrapped inside a `data` key.

### Test Case 2.2: Create a New Course (Positive)
* **Method:** `POST`
* **URL:** `{{baseUrl}}/api/courses`
* **Authorization:** `Bearer Token`
* **Body (raw JSON):**
  ```json
  {
    "name": "Advanced Laravel 10 Development",
    "price": 9900,
    "description": "Mastering advanced architecture design patterns.",
    "category_id": 1,
    "teacher_id": 1
  }
  ```
* **Expected Result:** `201 Created` | JSON object of the newly stored course containing its newly assigned `id`.

### Test Case 2.3: Course Creation - Missing Required Fields (Negative)
* **Method:** `POST`
* **URL:** `{{baseUrl}}/api/courses`
* **Body (raw JSON):** `{}`
* **Expected Result:** `422 Unprocessable Content` | Validation error JSON mapping out that `name` and `price` fields are mandatory.

### Test Case 2.4: Course Creation - Invalid Data Types & Missing Relations (Negative)
* **Method:** `POST`
* **URL:** `{{baseUrl}}/api/courses`
* **Body (raw JSON):**
  ```json
  {
    "name": "PHP Course",
    "price": "nine thousand", 
    "category_id": 99999,
    "teacher_id": 99999
  }
  ```
* **Expected Result:** `422 Unprocessable Content` | Validation failure due to `price` not being an integer. 
* *Note:* If validation rules for `category_id` and `teacher_id` are absent in your `CourseRequest`, this payload might cause a `500 Internal Server Error` due to an SQL Foreign Key Constraint violation.

### Test Case 2.5: Get a Single Course (Positive)
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/courses/1` (assuming ID 1 exists)
* **Expected Result:** `200 OK` | JSON object wrapping the requested course row.

### Test Case 2.6: Get Course - Non-Existent ID (Negative)
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/courses/999999`
* **Expected Result:** `404 Not Found` | Handled via Laravel's automatic Implicit Model Binding exception handler.

### Test Case 2.7: Update a Course (Positive)
* **Method:** `PUT`
* **URL:** `{{baseUrl}}/api/courses/1`
* **Body (raw JSON):**
  ```json
  {
    "name": "Completely Updated Course Title",
    "price": 12500
  }
  ```
* **Expected Result:** `200 OK` | JSON response payload representing the freshly updated record state.

### Test Case 2.8: Update Course - Invalid Data Inputs (Negative)
* **Method:** `PUT`
* **URL:** `{{baseUrl}}/api/courses/1`
* **Body (raw JSON):**
  ```json
  {
    "name": "",
    "price": -500
  }
  ```
* **Expected Result:** `422 Unprocessable Content` | Form request rule interception rejecting the empty name and out-of-bounds integer structure.

### Test Case 2.9: Delete a Course (Positive)
* **Method:** `DELETE`
* **URL:** `{{baseUrl}}/api/courses/1`
* **Expected Result:** `204 No Content` | Completely empty response body indicating successful execution.

### Test Case 2.10: Delete Course - Idempotency Check / Non-Existent ID (Negative)
* **Method:** `DELETE`
* **URL:** `{{baseUrl}}/api/courses/999999` (or repeating the exact deletion request on ID 1 right after Test Case 2.9)
* **Expected Result:** `404 Not Found` | The record is absent from the DB, so it cannot be matched for deletion.

---

## 3. Categories Resource (`/api/categories`)

### Test Case 3.1: Retrieve All Categories (Positive)
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/categories`
* **Expected Result:** `200 OK` | A JSON array containing all category models nested under a `data` parent key.

### Test Case 3.2: Create a New Category (Positive)
* **Method:** `POST`
* **URL:** `{{baseUrl}}/api/categories`
* **Body (raw JSON):**
  ```json
  {
    "name": "DevOps & Infrastructure"
  }
  ```
* **Expected Result:** `201 Created` | Returns the freshly initialized category entry with its generated ID.

### Test Case 3.3: Category Creation - String Character Limit Exceeded (Negative)
* **Method:** `POST`
* **URL:** `{{baseUrl}}/api/categories`
* **Body (raw JSON):** Send a name payload with a string length exceeding 255 characters.
* **Expected Result:** `422 Unprocessable Content` | Validation response stating: `"The name field must not be greater than 255 characters."`

### Test Case 3.4: Get a Single Category (Positive)
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/categories/1`
* **Expected Result:** `200 OK` | JSON details of the specified category element.

### Test Case 3.5: Get Category - Malicious Input / SQL Injection Check (Negative)
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/categories/1' OR 1=1 --`
* **Expected Result:** `404 Not Found` 
* *SQL Security Check:* Safe. Eloquent uses PDO parameter binding. It evaluates this entire rogue string literal as a search criteria look-up for a physical match rather than parsing it as active SQL operations.

### Test Case 3.6: Update a Category (Positive)
* **Method:** `PUT`
* **URL:** `{{baseUrl}}/api/categories/1`
* **Body (raw JSON):**
  ```json
  {
    "name": "Refactored Category Title"
  }
  ```
* **Expected Result:** `200 OK` | The entity reflects the updated string property.

### Test Case 3.7: Delete a Category & Cascade Database Check (Positive)
* **Method:** `DELETE`
* **URL:** `{{baseUrl}}/api/categories/1`
* **Expected Result:** `204 No Content`
* *Database Cascade Verification:* Execute `GET {{baseUrl}}/api/courses`. Check that all course records that were previously assigned to `category_id: 1` have been deleted by the database engine (`onDelete('cascade')`).

---

## 4. Teachers Resource (`/api/teachers`)

### Test Case 4.1: Retrieve All Teachers (Positive)
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/teachers`
* **Expected Result:** `200 OK` | A JSON collection layout wrapping all active teacher rows.

### Test Case 4.2: Create a New Teacher (Positive)
* **Method:** `POST`
* **URL:** `{{baseUrl}}/api/teachers`
* **Body (raw JSON):**
  ```json
  {
    "name": "Jane Doe",
    "description": "Expert Cloud Infrastructure Specialist."
  }
  ```
* **Expected Result:** `201 Created` | Successful instantiation with the supplied name and description.

### Test Case 4.3: Teacher Creation - Broken Content Formatting (Negative)
* **Method:** `POST`
* **URL:** `{{baseUrl}}/api/teachers`
* **Body (raw JSON):** Send a broken raw text body payload lacking standard JSON formatting symbols (e.g., `{ "name": "Missing Brace"`).
* **Expected Result:** `400 Bad Request` | Handled by Laravel's global middleware parser rejecting ill-formed JSON syntax payloads.

### Test Case 4.4: Get a Single Teacher (Positive)
* **Method:** `GET`
* **URL:** `{{baseUrl}}/api/teachers/1`
* **Expected Result:** `200 OK` | Object payload describing the matching teacher.

### Test Case 4.5: Update a Teacher (Positive)
* **Method:** `PUT`
* **URL:** `{{baseUrl}}/api/teachers/1`
* **Body (raw JSON):**
  ```json
  {
    "name": "Professor Jane Doe"
  }
  ```
* **Expected Result:** `200 OK` | Updates the instructor row attributes safely.

### Test Case 4.6: Delete a Teacher & Cascade Verification (Positive)
* **Method:** `DELETE`
* **URL:** `{{baseUrl}}/api/teachers/1`
* **Expected Result:** `204 No Content`
* *Database Integrity Verification:* Confirm via DB query or Postman that all course entries pointing to `teacher_id: 1` are dropped automatically from the system, preventing data anomalies.
