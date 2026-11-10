# University Management System — Lab 8

**Author:** Saduakas Bakdaulet (individual project)

A REST API for managing university students, courses, and grades. The application is built with Java Spring Boot and stores data in three separate PostgreSQL databases.

## Technology stack

- Java 17+ and Spring Boot 3.5
- Spring Web, Spring Data JPA, Hibernate
- PostgreSQL
- Maven
- Jakarta Bean Validation
- JUnit 5 and Mockito
- Postman for API testing

## Architecture

The application follows the Controller → Service → Repository structure.

| Database | Table | Responsibility |
|---|---|---|
| `student_db` | `students` | Student profiles |
| `course_db` | `courses` | Course catalog |
| `grade_db` | `grades` | Student grades |

Each database has its own `DataSource`, `EntityManagerFactory`, and `TransactionManager`. JPA repositories are configured separately for each persistence unit.

The transcript endpoint combines data from all three databases. Student and course references in the grades database are validated at the application level because they cannot use ordinary cross-database foreign keys.

## Features

- CRUD operations for students, courses, and grades
- Unique student emails and course codes
- Grade validation (0–100) and automatic letter-grade conversion
- Student transcript with course details and average score
- Structured API error responses using `@RestControllerAdvice`

Letter grades: A (90–100), B (80–89), C (70–79), D (60–69), F (0–59). The average is calculated from numeric scores, not as GPA or a credit-weighted result.

## Local setup

**Prerequisites:** JDK 17 or later, PostgreSQL, and Maven (or IntelliJ IDEA with Maven support).

1. Create the PostgreSQL databases `student_db`, `course_db`, and `grade_db`. See `sql/01-create-databases.sql`.
2. Configure the connection credentials as environment variables. The default PostgreSQL user is `postgres` and the port is `5432`.
3. Run `UniversityManagementApplication` from IntelliJ IDEA, or use `mvn spring-boot:run`.
4. The REST API is available at `http://localhost:8080`. Hibernate creates or updates the tables on application startup.

Required environment variables (set their values locally, not in source control):

```text
STUDENT_DB_PASSWORD=<your PostgreSQL password>
COURSE_DB_PASSWORD=<your PostgreSQL password>
GRADE_DB_PASSWORD=<your PostgreSQL password>
```

Optional overrides: `STUDENT_DB_URL`, `COURSE_DB_URL`, `GRADE_DB_URL`, the corresponding `*_DB_USERNAME` variables, and `SERVER_PORT`.

## REST API

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/students` | List students |
| GET | `/api/students/{id}` | Get student |
| POST | `/api/students` | Create student |
| PUT | `/api/students/{id}` | Update student |
| DELETE | `/api/students/{id}` | Delete student |
| GET | `/api/courses` | List courses |
| GET | `/api/courses/{id}` | Get course |
| POST | `/api/courses` | Create course |
| PUT | `/api/courses/{id}` | Update course |
| DELETE | `/api/courses/{id}` | Delete course |
| GET | `/api/grades` | List grades |
| POST | `/api/grades` | Assign grade |
| GET | `/api/grades/student/{id}` | List grades for a student |
| PUT | `/api/grades/{id}` | Update grade |
| DELETE | `/api/grades/{id}` | Delete grade |
| GET | `/api/students/{id}/transcript` | Student transcript |

### Example: student transcript

```http
GET /api/students/1/transcript
```

Sample response from local testing (selected fields):

```json
{
  "studentId": 1,
  "studentName": "Aruzhan Sadykova",
  "major": "Computer Science",
  "courses": [
    {
      "courseId": 1,
      "courseName": "Spring Boot Development",
      "courseCode": "CS301",
      "credits": 6,
      "score": 87,
      "letterGrade": "B"
    },
    {
      "courseId": 2,
      "courseName": "Mathematics for CS",
      "courseCode": "MATH101",
      "credits": 4,
      "score": 85,
      "letterGrade": "B"
    }
  ],
  "averageScore": 86.00
}
```

The actual endpoint also includes grading timestamps for individual courses.

## Testing

Import `postman/Lab8_University_Management.postman_collection.json` into Postman. The collection contains scenarios grouped as `01 Happy path`, `02 Error checks`, and `03 Cleanup (optional)`.

Local verification results:

- Three PostgreSQL connections initialized successfully.
- Transcript endpoint returned HTTP 200 and a calculated average of 86.00.
- Error checks: 8 passed, 0 failed (HTTP 400, 404, and 409).
- Happy path: 12 passed, 1 failed due to an existing student email (HTTP 409 instead of HTTP 201 on repeated creation). A clean test dataset is needed for a repeat run.

Unit tests can be run with `mvn test`.

## Implementation notes

The three database connections use independent local transactions; the project does not implement distributed transactions. Logical references between databases are checked by the service layer. The `ddl-auto=update` schema strategy is intended for local laboratory development.

## Project structure

```text
src/main/java/kz/edu/universitymanagement/
  config/       PostgreSQL configurations
  student/      Student controller, service, entity and repository
  course/       Course controller, service, entity and repository
  grade/        Grade controller, service, entity and repository
  transcript/   Transcript generation
  dto/          Request and response models
  exception/    API exception handling
src/main/resources/
  application.properties
src/test/        Unit tests
sql/             SQL setup and schemas
postman/         API request collection
screenshots/     Local verification screenshots
```
