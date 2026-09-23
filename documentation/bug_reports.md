# API Defect Report & Test Case Traceability Matrix

This document consolidates all functional defects discovered during the API execution suite. Each bug report is mapped directly to its corresponding Test Case ID to maintain clear traceability between requirements, test specifications, and code failures.

---

## Traceability Summary Table

| Defect ID | Linked Test Case ID | Endpoint | HTTP Method | Root Cause | Severity |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **BUG-001** | Test Case 2.5 & 2.6 | `/api/courses/{id}` | `GET` | Missing `show` action method inside `CourseController` | **Major** |
| **BUG-002** | Test Case 3.4 & 3.5 | `/api/categories/{id}` | `GET` | Missing `show` action method inside `CategoryController` | **Major** |
| **BUG-003** | Test Case 4.4 | `/api/teachers/{id}` | `GET` | Missing `show` action method inside `TeacherController` | **Major** |

---

## Detailed Defect Reports

### Bug Report BUG-001: Course Single Resource Retrieval Failure
* **Linked Test Case:** `TestCase 2.5: Get a Single Course (Positive)` & `TestCase 2.6: Get Course - Non-Existent ID (Negative)`
* **Endpoint / Route:** `GET {{baseUrl}}/api/courses/{id}`

#### Description
The API routing architecture relies on `Route::apiResource('courses', CourseController::class);` which exposes individual item lookup endpoints. However, executing this lookup collapses at runtime because the mandatory handler method `show()` was omitted during the development of `CourseController`.

#### Environment
- **Web Server:** Laragon (Apache/Nginx local virtual host environment)
- **Framework Stack:** Laravel (Eloquent ORM engine layer)
- **API Client:** Postman App

#### Steps to Reproduce
1. Ensure the database contains active records via seed execution (e.g., target record `id: 1` is populated).
2. Configure mandatory headers inside Postman: `Accept: application/json` and `Authorization: Bearer {{bearerToken}}`.
3. Dispatch a `GET` request directed toward `http://coursesapi.test`.

#### Expected Behavior
The framework router should intercept the payload, safely resolve the structural data bindings, and deliver a clean `200 OK` status wrapper presenting the nested resource object matching the target entity constraints.

#### Actual Behavior
The execution stack drops execution context due to a runtime controller exception, issuing an unexpected **`500 Internal Server Error`** layout:
```json
{
    "message": "Method App\\Http\\Controllers\\Api\\CourseController::show does not exist.",
    "exception": "BadMethodCallException",
    "file": "C:\\laragon\\www\\coursesAPI\\vendor\\laravel\\framework\\src\\Illuminate\\Routing\\Controller.php"
}
```

#### Defect Severity & Priority
- **Severity:** Major (Breaks atomic read operation on core application entities)
- **Priority:** High (Blocks complete end-to-end integration and parameter validation testing cycles)

#### Suggested Resolution Patch
Declare and define the explicit model binding receiver logic wrapper inside `App\Http\Controllers\Api\CourseController.php`:
```php
public function show(Course $course)
{
    return response()->json(['data' => $course]);
}
```

---

### Bug Report BUG-002: Category Single Resource Retrieval Failure
* **Linked Test Case:** `TestCase 3.4: Get a Single Category (Positive)` & `TestCase 3.5: Get Category - Malicious Input / SQL Injection Check (Negative)`
* **Endpoint / Route:** `GET {{baseUrl}}/api/categories/{id}`

#### Description
The routing mechanism declares full CRUD operations for categories, but fetching individual record properties throws an exception. `CategoryController` does not implement the structural `show()` method invoked by the implicit resource binding.

#### Steps to Reproduce
1. Supply authorized header token credentials.
2. Execute a `GET` call targeted to `http://coursesapi.test`.

#### Expected Behavior
An HTTP `200 OK` code providing the corresponding single category model attributes.

#### Actual Behavior
An HTTP **`500 Internal Server Error`** payload containing:
```json
{
    "message": "Method App\\Http\\Controllers\\Api\\CategoryController::show does not exist.",
    "exception": "BadMethodCallException"
}
```

#### Defect Severity & Priority
- **Severity:** Major  
- **Priority:** High

#### Suggested Resolution Patch
Add the missing operation interface hook inside `App\Http\Controllers\Api\CategoryController.php`:
```php
public function show(Category $category)
{
    return response()->json(['data' => $category]);
}
```

---

### Bug Report BUG-003: Teacher Single Resource Retrieval Failure
* **Linked Test Case:** `TestCase 4.4: Get a Single Teacher (Positive)`
* **Endpoint / Route:** `GET {{baseUrl}}/api/teachers/{id}`

#### Description
Triggering a single profile call for instructors throws an active runtime failure. The implementation lacks the matching action method block expected by the underlying Laravel framework runtime.

#### Steps to Reproduce
1. Establish standard authorization parameters.
2. Send a `GET` request toward `http://coursesapi.test`.

#### Expected Behavior
Successful data marshalling resulting in a `200 OK` status array formatting.

#### Actual Behavior
An HTTP **`500 Internal Server Error`** response:
```json
{
    "message": "Method App\\Http\\Controllers\\Api\\TeacherController::show does not exist.",
    "exception": "BadMethodCallException"
}
```

#### Defect Severity & Priority
- **Severity:** Major  
- **Priority:** High

#### Suggested Resolution Patch
Integrate the target implicit model resolution contract block inside `App\Http\Controllers\Api\TeacherController.php`:
```php
public function show(Teacher \$teacher)
{
    return response()->json(['data' => \$teacher]);
}
```
