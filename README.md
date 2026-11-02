# Lab 8 — University Management System

Java Spring Boot + Spring Data JPA + **three separate PostgreSQL databases** + REST + Postman.

## 1. Что реализовано

- `student_db` / `students`: CRUD, уникальный email.
- `course_db` / `courses`: CRUD, уникальный courseCode.
- `grade_db` / `grades`: CRUD, проверка существования studentId/courseId, score от 0 до 100, letterGrade.
- `GET /api/students/{id}/transcript`: студент + предметы + оценки + средний балл из всех трёх баз.
- `@RestControllerAdvice`: HTTP 400/404/409; недоступность БД при типовых ошибках соединения — 503.
- Раздельные `DataSource`, `EntityManagerFactory`, `TransactionManager`, `@EnableJpaRepositories`.
- Constructor injection, `@Transactional(transactionManager = "...")`, DTO, unit-тесты.

По заданию Lab 8 нужен backend, **веб-интерфейс не требуется**. Проверяем через Postman.

## 2. Что установить на Windows

1. JDK **17 или 21** (Spring Boot 3.5.16, проект компилируется для Java 17).
2. IntelliJ IDEA, Maven (можно использовать встроенный Maven IntelliJ).
3. PostgreSQL 15+ и pgAdmin 4.
4. Postman.

## 3. Создать базы

В pgAdmin войдите в сервер PostgreSQL. Создайте БД `student_db`, `course_db` и `grade_db`.
Либо в Query Tool базы `postgres` выполните `sql/01-create-databases.sql` (если сервер
или клиент запускает всё в одной транзакции, выполняйте три CREATE DATABASE отдельно).

Hibernate настроен с `hibernate.hbm2ddl.auto=update` для **каждого** EntityManagerFactory:
он создаёт таблицы при первом старте. Дополнительно приложены SQL схемы, если преподаватель
просит показать SQL: `sql/02-...` на `student_db`, `sql/03-...` на `course_db`,
`sql/04-...` на `grade_db`. При запуске SQL схем вручную убедитесь, что они выполнены
**в соответствующей базе**, а не в `postgres`.

## 4. Установить параметры подключения БЕЗ паролей в GitHub

В IntelliJ: **Run → Edit Configurations → UniversityManagementApplication → Environment variables**.
Установите (замените `YOUR_PASSWORD` локальным паролем PostgreSQL):

```text
STUDENT_DB_PASSWORD=YOUR_PASSWORD;COURSE_DB_PASSWORD=YOUR_PASSWORD;GRADE_DB_PASSWORD=YOUR_PASSWORD
```

В Windows PowerShell можно запустить через `mvn spring-boot:run` после команд:
```powershell
$env:STUDENT_DB_PASSWORD = "YOUR_PASSWORD"
$env:COURSE_DB_PASSWORD  = "YOUR_PASSWORD"
$env:GRADE_DB_PASSWORD   = "YOUR_PASSWORD"
mvn spring-boot:run
```

По умолчанию все 3 базы на `localhost:5432`, пользователь `postgres`.
Если другие URL/имена пользователей, задайте `STUDENT_DB_URL`, `COURSE_DB_URL`, `GRADE_DB_URL`
и `STUDENT_DB_USERNAME`, `COURSE_DB_USERNAME`, `GRADE_DB_USERNAME`.
Названия переменных также есть в `application.properties.example` (это образец, не secrets-файл).

При старте сервер доступен по адресу `http://localhost:8080`.
Если при открытии корневой страницы `/` получаете **404**, это нормально:
проект предоставляет `/api/...`, а не главную HTML страницу.

## 5. API (основные обязательные запросы)

| HTTP | URL | Назначение |
|---|---|---|
| GET | `/api/students` | Все студенты |
| POST | `/api/students` | Создать студента |
| PUT | `/api/students/{id}` | Изменить студента |
| DELETE | `/api/students/{id}` | Удалить студента |
| GET | `/api/courses` | Все курсы |
| POST | `/api/courses` | Создать курс |
| PUT | `/api/courses/{id}` | Изменить курс |
| DELETE | `/api/courses/{id}` | Удалить курс |
| POST | `/api/grades` | Добавить оценку |
| GET | `/api/grades/student/{id}` | Оценки студента |
| GET | `/api/students/{id}/transcript` | Общая ведомость из трёх баз |

Дополнительно реализованы `GET /api/students/{id}`, `GET /api/courses/{id}`,
`GET /api/grades`, `GET /api/grades/{id}`, `PUT /api/grades/{id}`, `DELETE /api/grades/{id}`.

**POST /api/students**, Body → raw → JSON:
```json
{"firstName":"Aruzhan","lastName":"Sadykova","email":"aruzhan@example.com","major":"Computer Science","enrollmentYear":2024}
```
**POST /api/courses**:
```json
{"courseName":"Spring Boot Development","courseCode":"CS301","credits":5,"instructor":"Dr. Kassen"}
```
**POST /api/grades** (подставьте реальные ID из POST ответов):
```json
{"studentId":1,"courseId":1,"score":95}
```
**GET /api/students/1/transcript** возвращает примерную структуру (для одной оценки):
```json
{
  "studentId":1,
  "studentName":"Aruzhan Sadykova",
  "major":"Computer Science",
  "courses":[{"courseId":1,"courseName":"Spring Boot Development","courseCode":"CS301","credits":5,"score":95,"letterGrade":"A","gradedAt":"2026-11-03T00:00:00Z"}],
  "averageScore":95.00
}
```
Это **пример формата**; реальные ID, время и значения зависят от ваших данных.

## 6. Postman

Откройте Postman → **Import → File → `postman/Lab8_University_Management.postman_collection.json`**.
У коллекции переменная `baseUrl=http://localhost:8080`. Сначала выполняйте запросы в папке
**01 Happy path** сверху вниз (на чистых базах). Для повторного прохождения сперва выполните
**03 Cleanup (optional)** или поменяйте email и коды курсов. Скрипты POST автоматически сохраняют `studentId`, `courseId`, `course2Id`, `gradeId`, `grade2Id`
для следующих запросов. Папка **02 Error checks** содержит запросы для 400, 404, 409.
Для сдачи сделайте **свои реальные скриншоты** с работающего PostgreSQL+Postman и добавьте в `screenshots/`.
В архиве есть инструкция `screenshots/README.md`, а не вымышленные скриншоты.

## 7. Проверить код

В IntelliJ откройте папку через **Open** и дождитесь импорта Maven.
Выберите JDK 17+; запустите главный класс `UniversityManagementApplication`.
Для unit-тестов `mvn test` (тесты сервисов не требуют запущенного PostgreSQL).

## 8. Обработка ошибок и особенности

- **400 Bad Request**: score 150; неверный email; отсутствие обязательного поля.
- **404 Not Found**: студент/курс/оценка с таким ID отсутствует.
- **409 Conflict**: дубликат email / courseCode / одна оценка на пару student+course;
  также запрет удаления студента/курса, пока для него есть оценки.
- **503 Service Unavailable**: типовые ошибки недоступности PostgreSQL.
- Пороговые буквы — **наше соглашение**: A (90–100), B (80–89), C (70–79),
  D (60–69), F (0–59); PDF не задаёт собственную шкалу.
- `averageScore` — простое арифметическое среднее **score**, не GPA и не взвешенное по credits.
- В `grades` нет SQL `FOREIGN KEY` на `student_db`/`course_db`: это три независимые БД.
  Существование ссылок проверяет GradeService через два других репозитория.
- Три локальных `JpaTransactionManager` **не означают одну общую атомарную транзакцию**.
  Смена записей между базами и параллельные удаления могут создавать расхождения;
  распределённые транзакции по заданию не требуются.
- Структура данных намеренно использует `hibernate.hbm2ddl.auto=update` для лабораторной работы;
  для production выбирают управляемые миграции и более строгие гарантии консистентности.

## 9. Готовимся к защите

Откройте `docs/DEFENSE_RU.md`: там короткая речь, 12 вопросов и порядок демонстрации.
Распределение ролей приведено как **предложение**, а не утверждение о сделанной работе.

## 10. Что сдавать на GitHub

Исходники `src/`, `pom.xml`, SQL скрипты `sql/`, `application.properties.example`,
Postman collection `postman/`, README и настоящие скриншоты в `screenshots/`.
Папку `target/` и файлы с реальными паролями **не коммитить**.
