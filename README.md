# 5COSC022W – Smart Campus Sensor & Room Management API

**Module:** Client-Server Architectures (5COSC022W)  
**Student:** [Your Name]  
**University:** University of Westminster  
**Year:** 2025/26

---

## Overview

This project is a RESTful API built using **JAX-RS (Jersey 3.1)** and deployed on **Apache Tomcat 10+**. It simulates a "Smart Campus" infrastructure management system for the University of Westminster, enabling facilities managers and automated building systems to manage Rooms and Sensors across campus.

The API follows REST architectural principles including resource-based URIs, appropriate HTTP methods, meaningful status codes, and a logical resource hierarchy. All data is stored in-memory using `ConcurrentHashMap` and `ArrayList` — no database is used.

**Base URL:** `http://localhost:8080/SmartCampusAPI/api/v1`

---

## Technology Stack

| Technology | Version |
|---|---|
| Java | 11 |
| JAX-RS (Jakarta) | 3.x |
| Jersey (Implementation) | 3.1.10 |
| Jackson (JSON) | via Jersey Media |
| Apache Tomcat | 10+ |
| Build Tool | Maven |

---

## Project Structure

```
src/main/java/com/smartcampus/smartcampusapi/
├── SmartCampusApp.java                          # @ApplicationPath config
├── model/
│   ├── Room.java                                # Room POJO
│   ├── Sensor.java                              # Sensor POJO
│   └── SensorReading.java                       # SensorReading POJO
├── store/
│   └── DataStore.java                           # In-memory singleton data store
├── resources/
│   ├── DiscoveryResource.java                   # GET /api/v1
│   ├── RoomResource.java                        # /api/v1/rooms
│   ├── SensorResource.java                      # /api/v1/sensors
│   └── SensorReadingResource.java               # /api/v1/sensors/{id}/readings
├── exception/
│   ├── RoomNotEmptyException.java               # 409 Conflict
│   ├── RoomNotEmptyExceptionMapper.java
│   ├── LinkedResourceNotFoundException.java     # 422 Unprocessable Entity
│   ├── LinkedResourceNotFoundExceptionMapper.java
│   ├── SensorUnavailableException.java          # 403 Forbidden
│   ├── SensorUnavailableExceptionMapper.java
│   └── GlobalExceptionMapper.java               # 500 catch-all
└── filter/
    └── LoggingFilter.java                       # Request & response logging
```

---

## How to Build and Run

### Prerequisites
- Java JDK 11 or higher installed
- Apache Maven installed (or use NetBeans built-in Maven)
- Apache Tomcat 10+ installed

### Step 1 — Clone the Repository

```bash
git clone https://github.com/[your-username]/5COSC022W-Smart-Campus-API.git
cd 5COSC022W-Smart-Campus-API
```

### Step 2 — Build the Project

```bash
mvn clean install
```

This will generate `SmartCampusAPI-1.0-SNAPSHOT.war` inside the `target/` folder.

### Step 3 — Deploy to Tomcat

**Option A — NetBeans:**
1. Open the project in NetBeans
2. Right-click project → Run
3. Select your Tomcat 10 server instance
4. NetBeans will deploy automatically

**Option B — Manual:**
1. Copy `target/SmartCampusAPI-1.0-SNAPSHOT.war` to your Tomcat `webapps/` folder
2. Start Tomcat: `bin/startup.bat` (Windows) or `bin/startup.sh` (Linux/Mac)
3. Access the API at `http://localhost:8080/SmartCampusAPI-1.0-SNAPSHOT/api/v1`

### Step 4 — Verify It's Running

Open your browser or Postman and visit:
```
http://localhost:8080/SmartCampusAPI/api/v1
```
You should see a JSON discovery response.

---

## API Endpoints Reference

### Discovery
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1` | Returns API metadata and resource links |

### Rooms
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/rooms` | Get all rooms |
| POST | `/api/v1/rooms` | Create a new room |
| GET | `/api/v1/rooms/{roomId}` | Get a specific room |
| DELETE | `/api/v1/rooms/{roomId}` | Delete a room (fails if sensors exist) |

### Sensors
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/sensors` | Get all sensors |
| GET | `/api/v1/sensors?type=CO2` | Filter sensors by type |
| POST | `/api/v1/sensors` | Register a new sensor |

### Sensor Readings
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/sensors/{sensorId}/readings` | Get all readings for a sensor |
| POST | `/api/v1/sensors/{sensorId}/readings` | Add a new reading |

---

## Sample curl Commands

### 1. Get API Discovery Info
```bash
curl -X GET http://localhost:8080/SmartCampusAPI/api/v1 \
  -H "Accept: application/json"
```

### 2. Get All Rooms
```bash
curl -X GET http://localhost:8080/SmartCampusAPI/api/v1/rooms \
  -H "Accept: application/json"
```

### 3. Create a New Room
```bash
curl -X POST http://localhost:8080/SmartCampusAPI/api/v1/rooms \
  -H "Content-Type: application/json" \
  -d '{"id": "ENG-101", "name": "Engineering Lab", "capacity": 40}'
```

### 4. Create a New Sensor (linked to an existing room)
```bash
curl -X POST http://localhost:8080/SmartCampusAPI/api/v1/sensors \
  -H "Content-Type: application/json" \
  -d '{"id": "CO2-002", "type": "CO2", "status": "ACTIVE", "currentValue": 350.0, "roomId": "LIB-301"}'
```

### 5. Filter Sensors by Type
```bash
curl -X GET "http://localhost:8080/SmartCampusAPI/api/v1/sensors?type=Temperature" \
  -H "Accept: application/json"
```

### 6. Add a Sensor Reading
```bash
curl -X POST http://localhost:8080/SmartCampusAPI/api/v1/sensors/TEMP-001/readings \
  -H "Content-Type: application/json" \
  -d '{"value": 24.5}'
```

### 7. Get All Readings for a Sensor
```bash
curl -X GET http://localhost:8080/SmartCampusAPI/api/v1/sensors/TEMP-001/readings \
  -H "Accept: application/json"
```

### 8. Delete a Room (will fail if sensors are assigned)
```bash
curl -X DELETE http://localhost:8080/SmartCampusAPI/api/v1/rooms/ENG-101 \
  -H "Accept: application/json"
```

---

## Report — Answers to Coursework Questions

### Part 1 — Service Architecture & Setup

**Q: Explain the default lifecycle of a JAX-RS Resource class. Is a new instance created per request or is it a singleton?**

By default, JAX-RS creates a **new instance of each resource class for every incoming HTTP request**. This is known as the "per-request" lifecycle. The practical consequence of this is that instance variables inside a resource class cannot be used to store shared state — any data written to an instance variable will be lost after the request completes, since the object is discarded. This is why a separate singleton `DataStore` class using `ConcurrentHashMap` is essential. The `ConcurrentHashMap` is thread-safe, meaning multiple simultaneous requests can read and write to the shared data store without causing race conditions or data corruption. If a regular `HashMap` were used instead, concurrent modifications could cause inconsistent state or even exceptions.

**Q: Why is HATEOAS considered a hallmark of advanced RESTful design? How does it benefit client developers?**

HATEOAS (Hypermedia as the Engine of Application State) is the principle that API responses should include links to related actions and resources, rather than forcing clients to construct URLs manually. For example, when a client retrieves a Room, the response can include a link to that room's sensors. This means clients do not need to hard-code URL patterns — they simply follow the links provided. This approach reduces coupling between client and server: if the server changes its URL structure, clients that follow HATEOAS links do not break. Compared to relying on static documentation, HATEOAS makes the API self-describing and navigable, which significantly reduces integration effort and the risk of errors caused by outdated documentation.

---

### Part 2 — Room Management

**Q: When returning a list of rooms, what are the implications of returning only IDs versus full room objects?**

Returning only IDs is bandwidth-efficient and faster for large collections, but it forces the client to make additional HTTP requests for each room's details — known as the N+1 problem. This increases latency and server load. Returning full room objects in a single response is more expensive in terms of payload size but far more practical for clients that need to display or process room data immediately. The appropriate choice depends on context: for a dashboard listing hundreds of rooms, a lightweight summary response (ID + name) is preferable. For a small collection, returning full objects is more convenient. In this implementation, full objects are returned since the dataset is small and the client benefits from having all data in one request.

**Q: Is DELETE idempotent in your implementation? Justify your answer.**

Yes, DELETE is idempotent in this implementation. Idempotency means that making the same request multiple times produces the same server state as making it once. In this API, the first DELETE request on a valid room removes it from the data store and returns `204 No Content`. If the same DELETE request is sent again, the room no longer exists, so the server returns `404 Not Found`. Although the HTTP status code differs between the first and subsequent calls, the **state of the server remains the same** — the room is absent. This satisfies the definition of idempotency, which concerns server state rather than response codes. This behaviour protects against accidental duplicate requests from clients.

---

### Part 3 — Sensor Operations & Filtering

**Q: Explain the consequences if a client sends data in a format other than JSON when `@Consumes(MediaType.APPLICATION_JSON)` is specified.**

If a client sends a request with a `Content-Type` header other than `application/json` (for example `text/plain` or `application/xml`), JAX-RS will automatically reject the request before it reaches the resource method. The runtime returns an HTTP `415 Unsupported Media Type` response. This is handled entirely by the framework — no custom code is needed. This behaviour enforces a strict contract between client and server, ensuring that the message body parser (Jackson in this case) only receives data it can safely deserialise. Sending `application/xml` would not only be rejected but, if somehow processed, would cause a deserialisation failure since Jackson's JSON parser cannot interpret XML syntax.

**Q: Why is `@QueryParam` preferred over path-based filtering (e.g., `/sensors/type/CO2`) for filtering collections?**

Query parameters are semantically more appropriate for filtering, searching, and sorting because they are optional modifiers of a resource collection rather than identifiers of a specific resource. A path segment like `/sensors/type/CO2` implies that `type/CO2` is a distinct resource, which is architecturally misleading. Query parameters make it clear that the base resource is `/sensors` and the `type` parameter is simply narrowing the result set. Query parameters also compose naturally: `?type=CO2&status=ACTIVE` is clean and readable, whereas nesting multiple filters into a path quickly becomes unwieldy and ambiguous. REST conventions reserve path segments for resource identification and query strings for resource manipulation and filtering.

---

### Part 4 — Sub-Resources

**Q: Discuss the architectural benefits of the Sub-Resource Locator pattern.**

The Sub-Resource Locator pattern improves separation of concerns by delegating the handling of nested resources to dedicated classes. In this API, `SensorResource` handles the `/sensors` collection, and rather than defining `/sensors/{id}/readings` endpoints directly within it, it delegates to a `SensorReadingResource` class. This keeps each class focused on a single responsibility. As the API grows — for example, adding `/sensors/{id}/alerts` or `/sensors/{id}/calibrations` — each concern can be handled in its own class without making `SensorResource` excessively large. It also improves testability, since each sub-resource class can be unit tested in isolation. Compared to a monolithic controller class that defines every nested path, this pattern produces code that is easier to read, maintain, and extend.

---

### Part 5 — Error Handling

**Q: Why is HTTP 422 more semantically accurate than 404 when a referenced resource is missing inside a valid JSON payload?**

HTTP `404 Not Found` conventionally means that the **requested URL** does not exist — the resource being addressed by the endpoint cannot be found. HTTP `422 Unprocessable Entity` means the server understood the request and the payload was syntactically valid JSON, but the **semantic content** of the payload is invalid. When a client POSTs a new Sensor with a `roomId` that does not exist, the endpoint `/api/v1/sensors` is perfectly valid and found. The problem is not the URL — it is that the data inside the request references a non-existent entity. Using `422` communicates precisely that the issue lies within the request body's logic, not the URL structure, giving the client developer a much clearer signal about what needs to be corrected.

**Q: From a cybersecurity standpoint, explain the risks of exposing Java stack traces to external API consumers.**

Exposing raw Java stack traces to external clients poses several serious security risks. First, stack traces reveal the **internal package and class structure** of the application, giving an attacker a map of the codebase to target. Second, they disclose the **exact versions of libraries and frameworks** in use (e.g., Jersey 3.1.10, Jackson 2.x), allowing attackers to look up known CVEs (Common Vulnerabilities and Exposures) for those specific versions. Third, stack traces can reveal **file system paths**, **database query structures**, and **internal variable names**, all of which reduce the effort required to craft a targeted attack. Fourth, error messages within traces may expose **business logic details** that should remain server-side. The Global Exception Mapper in this API prevents all of this by catching every unexpected `Throwable` and returning only a generic `500 Internal Server Error` message with no internal details.

**Q: Why is it better to use JAX-RS filters for cross-cutting concerns like logging rather than inserting log statements in every resource method?**

Using filters for cross-cutting concerns like logging follows the **Don't Repeat Yourself (DRY)** principle. If logging were added manually to every resource method, the same boilerplate code would appear dozens of times across the codebase. This creates maintenance burden: if the log format needs to change, every method must be updated individually. A single `LoggingFilter` class implementing `ContainerRequestFilter` and `ContainerResponseFilter` centralises this logic in one place. Filters are also applied **consistently and automatically** to every request and response without any developer effort — no resource method can accidentally be missed. Furthermore, filters can be added, removed, or modified without touching any resource class, making the system more modular and easier to maintain.

---

## Pre-loaded Sample Data

The API comes with sample data for testing:

| Type | ID | Details |
|------|----|---------|
| Room | `LIB-301` | Library Quiet Study, capacity 50 |
| Room | `LAB-101` | Computer Lab, capacity 30 |
| Sensor | `TEMP-001` | Temperature, ACTIVE, in LIB-301 |
| Sensor | `CO2-001` | CO2, MAINTENANCE, in LAB-101 |

---

## Error Responses

| Status Code | Scenario |
|-------------|----------|
| 400 Bad Request | Missing required fields |
| 403 Forbidden | Posting a reading to a MAINTENANCE sensor |
| 404 Not Found | Room or sensor ID does not exist |
| 409 Conflict | Deleting a room that still has sensors |
| 415 Unsupported Media Type | Sending non-JSON content |
| 422 Unprocessable Entity | Creating a sensor with a non-existent roomId |
| 500 Internal Server Error | Any unexpected server-side error |
