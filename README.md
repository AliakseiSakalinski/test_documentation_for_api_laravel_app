# API Testing and Test Documentation

This repository serves as a QA Engineer project, containing a comprehensive suite of test documentation for an e-learning platform REST API (coursesAPI).

---

## Included Test Documentation Artifacts

*   Test Case Specification (./documentation/test_cases.md): A collection of 19 detailed positive and negative test cases covering Laravel Sanctum authentication protection, stateless CRUD operations, input type validations, character limits, SQL injection resilience, and database cascade integrity checks.
*   Defect Reports and Traceability Matrix (./documentation/bug_reports.md): Documentation of discovered critical runtime issues causing 500 Internal Server Error failures, mapped directly back to their originating Test Case IDs.
*   Postman Collection JSON (./postman/coursesAPI_collection.json): A structured collection file detailing all system endpoints, pre-configured for import. All base URLs and authorization keys are safely isolated into abstract environment variables.

---

## Test Coverage Scope

1. Security and Authentication: Verifying that protected private resource endpoints restrict unauthenticated requests with a 401 Unauthorized response, and checking framework parameters against SQL injection patterns.
2. Stateful CRUD Operations: Positive validation mapping the complete lifecycle of creating, reading, updating, and deleting records across core entities (Courses, Categories, Teachers).
3. Boundary Testing and Error Handling: Intentionally dispatching malformed payloads, out-of-bounds integers (e.g., string types for price values), empty mandatory values, and broken JSON text syntax to confirm accurate validation responses (400 Bad Request, 422 Unprocessable Content).
4. Data Integrity: Validating downstream constraints to confirm that database engines execute correct cascade deletion routines when parent structural rows are cleared.

---

## Defect Traceability Summary

During the execution cycle, three major defects were identified where the platform failed to implement single-resource lookup actions across controllers, resulting in unhandled 500 Internal Server Error responses.

| Defect ID | Linked Test Case ID | Target Endpoint | HTTP Method | Severity | Root Cause |
| :--- | :--- | :--- | :--- | :--- | :--- |
| BUG-001 | Test Case 2.5 & 2.6 | /api/courses/{id} | GET | Major | Missing show action method inside CourseController |
| BUG-002 | Test Case 3.4 & 3.5 | /api/categories/{id} | GET | Major | Missing show action method inside CategoryController |
| BUG-003 | Test Case 4.4 | /api/teachers/{id} | GET | Major | Missing show action method inside TeacherController |

Detailed replication instructions and recommended code resolution patches are provided in the full bug documentation directory.

---

## Review Guide

1. Navigate to the documentation directory to review the complete Test Cases and Bug Reports markdown files.
2. To examine the underlying API structure, download the raw file inside the postman folder and import it directly into your local Postman workspace application via the File Import wizard.
