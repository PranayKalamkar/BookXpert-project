Employee Management System — README

A full-stack Employee Management System (Intern assignment) — React + MUI frontend, .NET 8 Web API backend (ADO.NET + MySQL), stored procedures, report generation (PDF/Excel), charts (Recharts), and CI/CD. This README is detailed and prescriptive: setup, architecture, API reference, DB schema & stored procedures, build & run steps, troubleshooting with realistic stack traces and how to fix them, and recommended CI/CD + monitoring.

Table of contents

Project summary

Tech stack

Project layout (file structure)

Prerequisites (local machine)

Environment variables (frontend & backend)

Database schema & key stored procedures (SQL)

Backend — setup, build, run, tests

Frontend — setup, build, run

API Reference (routes, sample requests/responses)

Report generation (PDF/Excel) — how it works

Logging, monitoring & distributed tracing (recommended setup)

CI / CD (GitHub Actions example)

Troubleshooting — common errors + sample stack traces & fixes

Contribution & license

1. Project summary

This app manages employees — CRUD for employee records, attendance, salary, department analytics, and report generation. The frontend uses React + MUI v7 and Recharts. The backend is a stateless ASP.NET Core Web API (targeting .NET 8) using ADO.NET to call MySQL stored procedures. Reports are generated client-side using jspdf and xlsx or server-side if desired.

2. Tech stack

Frontend: React (V18+), TypeScript (optional), MUI v7 (Unstable_Grid2), Recharts, axios, jsPDF, xlsx, file-saver

Backend: .NET 8 (ASP.NET Core Web API), ADO.NET (MySql.Data or MySqlConnector), JWT authentication (optional), Serilog for logging, OpenTelemetry for tracing

Database: MySQL (8.x recommended) with stored procedures

CI/CD: GitHub Actions (build/test/deploy)

Monitoring: Application Insights / Prometheus + Grafana / ELK stack

Local dev: node 18+, npm/yarn, .NET 8 SDK, MySQL

3. Project layout (suggested)
/ (repo root)
├─ backend/
│  ├─ EmployeeApi.sln
│  ├─ src/
│  │  ├─ EmployeeApi/ (ASP.NET Core project)
│  │  │  ├─ Controllers/
│  │  │  ├─ Repositories/
│  │  │  ├─ Services/
│  │  │  ├─ Models/
│  │  │  ├─ appsettings.Development.json
│  │  │  └─ Program.cs
├─ frontend/
│  ├─ package.json
│  ├─ src/
│  │  ├─ components/
│  │  │  ├─ Report.tsx
│  │  │  └─ Dashboard/
│  │  ├─ contexts/
│  │  └─ utils/
├─ db/
│  ├─ schema.sql
│  └─ stored_procedures.sql
├─ .github/
│  └─ workflows/
│     └─ ci-cd.yml
└─ README.md

4. Prerequisites

Node.js 18+ and npm 9+ or yarn

.NET 8 SDK installed (dotnet --version -> 8.x)

MySQL server (8.x) or Docker image

Git and GitHub account (for CI/CD examples)

Optional tools:

Postman/Insomnia for API testing

VS Code / Visual Studio for development

5. Environment variables
Backend (EmployeeApi/appsettings.* or env)
# Connection
DB__Host=localhost
DB__Port=3306
DB__Database=employee_db
DB__User=root
DB__Password=your_db_password

# JWT (if enabled)
JWT__Issuer=your-issuer
JWT__Audience=your-audience
JWT__Key=super-secret-key (store securely in secrets manager)

# Logging/Monitoring
SERILOG__WRITE_TO__CONSOLE__ENABLED=true
OTEL__SERVICE_NAME=EmployeeApi

Frontend (.env)
REACT_APP_API_BASE_URL=http://localhost:5000/api
REACT_APP_ENV=development

6. Database schema & key stored procedures
Example schema (db/schema.sql)
CREATE DATABASE IF NOT EXISTS employee_db;
USE employee_db;

CREATE TABLE tbl_employees (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(150) NOT NULL,
  email VARCHAR(150) UNIQUE NOT NULL,
  department VARCHAR(100),
  salary DECIMAL(12,2),
  join_date DATE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE tbl_attendance (
  id INT AUTO_INCREMENT PRIMARY KEY,
  employee_id INT NOT NULL,
  attendance_date DATE NOT NULL,
  status ENUM('present','absent','leave') DEFAULT 'present',
  FOREIGN KEY (employee_id) REFERENCES tbl_employees(id) ON DELETE CASCADE
);

Example stored procedures (db/stored_procedures.sql)
DELIMITER $$

CREATE PROCEDURE GetAllEmployees()
BEGIN
  SELECT id, name, email, department, salary, join_date FROM tbl_employees;
END$$

CREATE PROCEDURE GetEmployeeById(IN pid INT)
BEGIN
  SELECT id, name, email, department, salary, join_date FROM tbl_employees WHERE id = pid;
END$$

CREATE PROCEDURE AddEmployee(
  IN p_name VARCHAR(150),
  IN p_email VARCHAR(150),
  IN p_department VARCHAR(100),
  IN p_salary DECIMAL(12,2),
  IN p_join_date DATE
)
BEGIN
  INSERT INTO tbl_employees(name,email,department,salary,join_date)
  VALUES(p_name,p_email,p_department,p_salary,p_join_date);
  SELECT LAST_INSERT_ID() as newId;
END$$

CREATE PROCEDURE UpdateEmployee(
  IN p_id INT,
  IN p_name VARCHAR(150),
  IN p_email VARCHAR(150),
  IN p_department VARCHAR(100),
  IN p_salary DECIMAL(12,2),
  IN p_join_date DATE
)
BEGIN
  UPDATE tbl_employees
  SET name=p_name, email=p_email, department=p_department, salary=p_salary, join_date=p_join_date
  WHERE id=p_id;
  SELECT ROW_COUNT() as rowsAffected;
END$$

CREATE PROCEDURE DeleteEmployee(IN p_id INT)
BEGIN
  DELETE FROM tbl_employees WHERE id = p_id;
  SELECT ROW_COUNT() as rowsAffected;
END$$

DELIMITER ;

7. Backend — setup, build, run, tests
Setup

cd backend/src/EmployeeApi

dotnet restore

Create appsettings.Development.json with DB connection strings (or use environment variables).

Local run
dotnet run --project ./EmployeeApi.csproj
# or
dotnet watch run


API default URL: https://localhost:5001 (Kestrel)

Important server-side patterns

Use IDbConnection factory with MySqlConnector or MySql.Data in Program.cs.

Use using / await correctly for ADO.NET calls.

Centralize error handling with a middleware to return consistent JSON errors. Example JSON error:

{
  "status": 500,
  "error": "InternalServerError",
  "message": "An unexpected error occurred"
}

Unit / Integration tests

Use xUnit + Moq for service layer + repository unit tests.

For integration tests, use a test MySQL instance (Docker) and run stored procedures in a test DB.

8. Frontend — setup, build, run
Install
cd frontend
npm ci
# or
yarn install

Run (dev)
npm start
# opens at http://localhost:3000

Build
npm run build
# artifacts in /build

MUI v7 Grid gotcha

MUI v7 uses Unstable_Grid2. Import like:

import Grid from '@mui/material/Unstable_Grid2';


Use breakpoints directly on Grid:

<Grid container spacing={2}>
  <Grid xs={12} sm={6} md={3}>...</Grid>
</Grid>

Report generation libraries used

jspdf + jspdf-autotable

xlsx for Excel creation

file-saver to download files

9. API Reference (sample)
Auth

POST /api/auth/login
Body: { "username": "...", "password": "..." }
Response: { "token": "JWT", "expiresIn": 3600 }

Employees

GET /api/employees — returns list of employees

GET /api/employees/{id} — employee by id

POST /api/employees — create employee
Body: { name, email, department, salary, joinDate }

PUT /api/employees/{id} — update employee

DELETE /api/employees/{id} — delete employee

Reports (frontend handles PDFs, but backend may expose analytics)

GET /api/reports/hiring-trends — { month: "Jan", hires: 5 }[]

GET /api/reports/department-growth — { dept: "Engineering", count: 12 }[]

GET /api/reports/attendance — { name: "John", days: 20 }[]

Sample curl
curl -X GET "http://localhost:5000/api/employees" -H "Authorization: Bearer <token>"

10. Report generation (how it works)

Directory, Attendance, Salary, Performance reports can be generated on the client using employee data and aggregated data from mockData or API endpoints.

PDF: jsPDF + autoTable to build tables and call doc.save('filename.pdf').

Excel: xlsx → XLSX.utils.json_to_sheet(data) → XLSX.write(wb, { type: 'array', bookType: 'xlsx' }) → file-saver.

If you prefer server-side PDFs, consider DinkToPdf or headless Chrome (Puppeteer) in a backend job.

11. Logging, monitoring & distributed tracing (recommended)

Logging: Serilog with sinks (console, file, Seq). Configure structured logs in Program.cs:

Log.Logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();


Monitoring: Application Insights (Azure) or Prometheus + Grafana (containers).

Tracing: OpenTelemetry for .NET; export traces to Jaeger/Zipkin or Azure Monitor.

12. CI / CD (GitHub Actions example)

/.github/workflows/ci-cd.yml (high level)

name: CI

on: [push, pull_request]

jobs:
  build-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      - name: Restore & Build
        run: dotnet build ./backend/src/EmployeeApi.sln --configuration Release
      - name: Run Tests
        run: dotnet test ./backend/tests --no-build --verbosity normal

  build-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: cd frontend && npm ci
      - run: cd frontend && npm run build

  deploy:
    needs: [build-backend, build-frontend]
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Azure Web App (example)
        uses: azure/webapps-deploy@v4
        with:
          app-name: my-app
          package: backend/src/EmployeeApi/bin/Release/net8.0/publish

13. Troubleshooting — common errors + stack traces & fixes

Below are realistic error examples you may encounter — each with an explanation and steps to fix.

Error: MUI Grid TypeScript overload (example)

Symptom: No overload matches this call when using <Grid item xs={12} ...>.

Cause: Using MUI v7 with @mui/material/Grid import (old API). MUI v7 uses Unstable_Grid2.

Fix:

// Replace this:
import Grid from '@mui/material/Grid';

// With this:
import Grid from '@mui/material/Unstable_Grid2';


Explanation: Grid v2 in v7 exposes responsive props directly (no item/container props). Using the legacy Grid type conflicts with TypeScript signatures.

Example .NET stack trace (backend exception)
System.NullReferenceException: Object reference not set to an instance of an object.
   at EmployeeApi.Repositories.EmployeeRepository.GetEmployeeById(Int32 id) in /src/Repositories/EmployeeRepository.cs:line 45
   at EmployeeApi.Controllers.EmployeeController.GetById(Int32 id) in /src/Controllers/EmployeeController.cs:line 27
   at Microsoft.AspNetCore.Mvc.Infrastructure.ActionMethodExecutor.AwaitableObjectResultExecutor.Execute(ActionContext actionContext, IActionResultTypeMapper mapper, ObjectMethodExecutor executor, Object controllerContext)


How to read:

The exception type: System.NullReferenceException — trying to access a property on null.

The topmost app frame tells you where: EmployeeRepository.GetEmployeeById line 45. Inspect line 45.

Typical causes and fixes:

Cause: reader["col"] returning DBNull or a null value; code directly casts to string/int.

Fix: Null-check results before use, e.g.:

var emailObj = reader["email"];
var email = emailObj == DBNull.Value ? string.Empty : Convert.ToString(emailObj);


Cause: A dependency (e.g., _databaseConnection) not injected. Fix DI registration in Program.cs.

Prevention:

Use safe mappings from DB to models.

Add guard clauses; centralize mapping code.

Example JS error (frontend)
TypeError: Cannot read property 'map' of undefined
    at Report.tsx:123
    at renderWithHooks (react-dom.development.js:14985)
    at mountIndeterminateComponent


Cause: You're calling .map on undefined (e.g., employees may be undefined before API response).

Fix:

Provide safe defaults: const employees = employeesState ?? [];

Render conditionally:

{employees?.length ? employees.map(e => <Row .../>) : <EmptyState />}

Database error: Stored procedure not found

Error:

MySqlException: Procedure employee_db.GetAllEmployes does not exist


Cause: Typo in stored procedure name or the procedure wasn't created in the DB you connected to.

Fix:

Confirm DB selection in connection string: Database=employee_db;

Run SHOW PROCEDURE STATUS WHERE Db = 'employee_db';

Recreate GetAllEmployees and ensure the name matches exactly.

14. Contribution & License

Contributions: open an issue describing the feature or bug; submit PRs to develop branch.

Code style: Prettier for frontend, EditorConfig + dotnet format for backend.

License: MIT (or your preferred license).

Final notes — Step-by-step checklist to get running locally

Clone repo:

git clone https://github.com/<your-repo>.git
cd <your-repo>


Prepare DB:

Start MySQL (or docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 -d mysql:8)

Run SQL scripts:

mysql -u root -p < db/schema.sql
mysql -u root -p < db/stored_procedures.sql


Start backend:

cd backend/src/EmployeeApi
dotnet restore
dotnet run


Start frontend:

cd frontend
npm ci
npm start


Test endpoints (Postman):

GET http://localhost:5000/api/employees

Generate reports from UI (Reports page).
