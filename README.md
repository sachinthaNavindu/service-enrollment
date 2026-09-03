# Enrollment Service

A Spring Boot microservice responsible for managing student enrollments and providing enrollment information through a RESTful API.

The service persists enrollment data in MySQL and communicates with the **Student Service** to retrieve student information when returning enrollment details.

---

## Overview

The Enrollment Service provides APIs to:

* Create enrollments
* Retrieve all enrollments
* Retrieve enrollments by program
* Retrieve an enrollment by ID
* Update enrollments
* Delete enrollments
* Enrich enrollment responses with student information

It is designed to operate as part of a distributed microservices architecture using **Spring Cloud Config** and **Netflix Eureka**.

---

## Architecture

```text
                         ┌──────────────────┐
                         │   API Gateway    │
                         │      :7000       │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ Enrollment      │
                         │ Service :8002   │
                         └───────┬──────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
                    ▼                         ▼
             ┌──────────────┐          ┌──────────────┐
             │    MySQL     │          │    Student   │
             │   Database   │          │    Service   │
             │    :14500    │          │    :8000     │
             └──────────────┘          └──────────────┘

                         ▲
                         │
                ┌────────┴────────┐
                │                 │
         ┌──────────────┐  ┌──────────────┐
         │ Config Server│  │ Eureka       │
         │    :9000     │  │ Registry     │
         │              │  │    :9001     │
         └──────────────┘  └──────────────┘
```

---

## Technology Stack

| Technology                 | Purpose                      |
| -------------------------- | ---------------------------- |
| Java 25                    | Application runtime          |
| Spring Boot 4.1.0          | Application framework        |
| Spring Cloud 2025.1.2      | Microservices infrastructure |
| Spring Data JPA            | Data persistence             |
| MySQL                      | Relational database          |
| Spring RestClient          | Inter-service communication  |
| MapStruct                  | DTO and entity mapping       |
| Lombok                     | Boilerplate reduction        |
| Spring Validation          | Request validation           |
| Netflix Eureka Client      | Service discovery            |
| Spring Cloud Config Client | Centralized configuration    |
| Spring Boot Actuator       | Health and monitoring        |
| Maven                      | Build management             |

---

## Service Details

| Property      | Value                 |
| ------------- | --------------------- |
| Service       | Enrollment Service    |
| Port          | `8002`                |
| Group ID      | `lk.ijse.eca`         |
| Artifact ID   | `Enrollment-Service`  |
| Database      | MySQL                 |
| Database Port | `14500`               |
| API Base Path | `/api/v1/enrollments` |

The service configuration uses a MySQL database named `eca` on port `14500`.

---

## API Endpoints

### Create Enrollment

```http
POST /api/v1/enrollments
```

Creates a new student enrollment.

**Request Body**

```json
{
  "date": "2025-01-15",
  "studentId": "123456789V",
  "programId": "DEVOPS"
}
```

---

### Get All Enrollments

```http
GET /api/v1/enrollments
```

Returns all enrollments.

---

### Get Enrollments by Program

```http
GET /api/v1/enrollments?programId={id}
```

Returns enrollments associated with a specific program.

---

### Get Enrollment by ID

```http
GET /api/v1/enrollments/{id}
```

Returns a specific enrollment using its ID.

---

### Update Enrollment

```http
PUT /api/v1/enrollments/{id}
```

Updates an existing enrollment.

**Request Body**

```json
{
  "date": "2025-02-01",
  "studentId": "123456789V",
  "programId": "BSC"
}
```

---

### Delete Enrollment

```http
DELETE /api/v1/enrollments/{id}
```

Deletes an existing enrollment.

The enrollment ID is an auto-generated `Long` value.

---

## Response Example

A successful enrollment response can include the associated student information retrieved from the Student Service.

```json
{
  "id": 1,
  "date": "2026-01-11",
  "studentId": "123456789V",
  "programId": "DEVOPS",
  "student": {
    "name": "Ethan Hewage",
    "address": "123 Main Street, Colombo",
    "mobile": "0771234567",
    "email": "Ethan@example.com"
  }
}
```

The `student` information is populated through communication with the Student Service.

---

## Service-to-Service Communication

The Enrollment Service communicates with the **Student Service** using Spring `RestClient`.

```text
Enrollment Service
        │
        │ Request student details
        ▼
  Student Service
        │
        ▼
 Student Information
```

This keeps student data managed by the Student Service while allowing enrollment responses to provide the required student details.

---

## Service Discovery

The service is registered with **Netflix Eureka**.

Instead of relying on fixed service addresses, the Enrollment Service can participate in service discovery within the microservices environment.

```text
Enrollment Service
        │
        ▼
Eureka Service Registry
        │
        ▼
Student Service
```

The Service Registry runs on:

```text
http://localhost:9001
```

---

## Centralized Configuration

The Enrollment Service uses **Spring Cloud Config Client** to retrieve its configuration from the centralized Config Server.

```text
Enrollment Service
        │
        │ Configuration
        ▼
 Config Server
     :9000
```

The Config Server should be available before starting the Enrollment Service.

---

## Prerequisites

Before running the service, ensure the following are available:

* Java 25
* MySQL
* Git
* Config Server
* Service Registry
* Student Service

The MySQL instance must be accessible on port:

```text
14500
```

The repository's documented startup dependencies include Config Server, Service Registry, API Gateway, and Student Service.

---

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/sachinthaNavindu/service-enrollment.git
```

Navigate to the project:

```bash
cd service-enrollment
```

---

## Run the Application

Using Maven Wrapper:

### Linux / macOS

```bash
./mvnw spring-boot:run
```

### Windows

```bash
mvnw.cmd spring-boot:run
```

The service will start on:

```text
http://localhost:8002
```

---

## Build the Application

```bash
./mvnw clean package
```

For Windows:

```bash
mvnw.cmd clean package
```

The generated JAR file will be available in:

```text
target/
```

Run the packaged application with:

```bash
java -jar target/Enrollment-Service-*.jar
```

---

## Startup Order

For the complete microservices environment, use the following startup order:

```text
1. Config Server       :9000
        ↓
2. Service Registry    :9001
        ↓
3. API Gateway         :7000
        ↓
4. Student Service     :8000
        ↓
5. Program Service     :8001
        ↓
6. Enrollment Service  :8002
```

This ensures that configuration, service discovery, and required dependent services are available before the Enrollment Service starts.

---

## Health Check

Spring Boot Actuator is included for application monitoring.

Once the service is running:

```text
http://localhost:8002/actuator/health
```

Use this endpoint to verify the application's health status.

---

## Testing

The API can be tested using tools such as:

* Postman
* cURL
* Swagger/OpenAPI clients
* Frontend applications through the API Gateway

The repository also provides a Postman collection for testing the Enrollment Service endpoints.

---

## Project Structure

```text
service-enrollment/
│
├── .mvn/
│   └── wrapper/
│
├── src/
│   └── main/
│       ├── java/
│       │   └── lk/
│       │       └── ijse/
│       │           └── eca/
│       │               └── enrollment/
│       │
│       └── resources/
│
├── .gitattributes
├── .gitignore
├── mvnw
├── mvnw.cmd
├── pom.xml
└── README.md
```

---

## Microservices Integration

The Enrollment Service is part of a larger microservices architecture:

```text
                     API Gateway
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
      Student         Program       Enrollment
      Service         Service        Service
          │              │              │
          └──────────────┼──────────────┘
                         │
                   Service Registry
                         │
                   Config Server
```

The Enrollment Service specifically depends on the Student Service when student information needs to be included in enrollment responses.

---

## Author

**Sachintha Navindu**
