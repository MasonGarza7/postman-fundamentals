# Postman Fundamentals

A Postman testing suites covering HTTP fundamentals, authentication, CRUD operations, schema validation, an data-driven testing with Newman CLI integration.

Built as a structured example project to develop and demonstrate proficiency in API testing for software quality assurance.



## Table of Contents

- [Overview](#overview)
- [Tooling](#tools--technologies)
- [Project Structure](#project-structure)
- [Collections](#collections)
- [How to Run](#how-to-run)
- [Concepts Covered](#concepts-covered)
  - [HTTP Fundamentals](#http-fundamentals)
  - [Authentication](#authentication)
  - [CRUD Operations](#crud-operations)
  - [Test Scripting](#test-scripting)
  - [Variables & Chaining](#variables--chaining)
  - [Data-Driven Testing](#data-driven-testing)
  - [Newman & CI/CD](#newman--cicd)
- [Cheat Sheet](#cheat-sheet)



## Overview

This project covers two practice APIs:

| API | Purpose |
|-----|---------|
| [the-internet.herokuapp.com](https://the-internet.herokuapp.com) | HTTP status codes, authentication patterns |
| [restful-booker.herokuapp.com](https://restful-booker.herokuapp.com) | Full REST API with JSON, CRUD, and token auth |

All tests are written in Postman and runnable via Newman CLI, making them portable and CI/CD ready.



## Tooling

- **Postman**: API client for building and running requests
- **Newman**: Postman's CLI runner for automated/CI execution
- **Node.js**: Required runtime for Newman
- **JavaScript**: Used in Postman test scripts (`pm.*` API)



## Project Structure

```
postman-fundamentals/
├── collections/
│   ├── the-internet-collection.json          # HTTP & auth tests
│   ├── restful-booker-ui-collection.json     # CRUD tests (hardcoded, UI-friendly)
│   └── restful-booker-newman-collection.json # CRUD tests (CSV data-driven, Newman)
├── environments/
│   ├── the-internet-dev.json                 # Base URL for the-internet
│   └── restful-booker-dev.json               # Base URL for Restful Booker
├── data/
│   └── bookings.csv                          # Test data for data-driven runs
├── .gitignore
├── package.json
└── README.md
```



## Collections

### the-internet Collection

Tests HTTP behavior and authentication patterns against [the-internet.herokuapp.com](https://the-internet.herokuapp.com).

| Request | Method | Endpoint | What's Tested |
|---------|--------|----------|---------------|
| 301 - Status Code Bug Documentation | GET | `/status_codes/301` | Documents broken redirect — missing `Location` header |
| Basic Auth - Happy Path | GET | `/basic_auth` | 200 with valid credentials |
| Basic Auth - Unauthorized | GET | `/basic_auth` | 401 + `WWW-Authenticate` header with invalid credentials |
| Form Auth - Happy Path | POST | `/authenticate` | 200 + success message in body |
| Form Auth - Unauthorized | POST | `/authenticate` | 200 + error message in body (status ≠ success) |

**Key concept demonstrated:** A `200` status code does not always mean success. Body assertions are required to verify actual outcomes.

---

### Restful Booker UI Collection

Full CRUD test suite against [restful-booker.herokuapp.com](https://restful-booker.herokuapp.com). Hardcoded values for use in Postman UI and standalone Newman runs.

| Request | Method | Endpoint | What's Tested |
|---------|--------|----------|---------------|
| GET All Bookings | GET | `/booking` | Returns JSON array, schema, response time |
| POST Create a Booking | POST | `/booking` | Creates booking, validates response schema, chains `bookingId` |
| GET Single Booking | GET | `/booking/{{bookingId}}` | Validates full booking schema |
| POST Get Auth Token | POST | `/auth` | Returns token, chains `authToken` |
| PUT Update a Booking | PUT | `/booking/{{bookingId}}` | Updates booking, validates changes |
| DELETE a Booking | DELETE | `/booking/{{bookingId}}` | Deletes booking, confirms 201 response |

---

### Restful Booker Newman Collection

Same as the UI collection but uses CSV variables in **POST Create a Booking** for data-driven runs. Intended for Newman with `--iteration-data`. Running collection runs with CSV data is a paid feature in the Postman UI, so I duplicated the project with the CSV modifcation as a workaround. 



## How to Run

### Prerequisites

1. Install [Node.js](https://nodejs.org/)
2. Install Newman:

```bash
npm install -g newman
```

Or install locally via package.json:

```bash
npm install
```

---

### Running with Newman

Run all collections:

```bash
npm run test:all
```

Or run individually:

```bash
# The Internet — HTTP & Auth tests
npm run test:internet

# Restful Booker — CRUD tests (hardcoded)
npm run test:booker:ui

# Restful Booker — CRUD tests (data-driven, 3 iterations)
npm run test:booker:data
```

Or run manually:

```bash
newman run collections/the-internet-collection.json -e environments/the-internet-dev.json
  ```

```bash
newman run collections/restful-booker-ui-collection.json -e environments/restful-booker-dev.json
```

```bash
newman run collections/restful-booker-newman-collection.json -e environments/restful-booker-dev.json --iteration-data data/bookings.csv
```


## Concepts Covered

### HTTP Fundamentals

Every HTTP interaction is a **request → response** cycle.

**Core HTTP methods:**

| Method | Purpose |
|--------|---------|
| `GET` | Retrieve data |
| `POST` | Create new resource |
| `PUT` | Replace existing resource |
| `PATCH` | Partially update resource |
| `DELETE` | Remove resource |

**Common status codes:**

| Code | Meaning |
|------|---------|
| `200` | OK |
| `201` | Created |
| `301` | Moved Permanently (redirect) |
| `400` | Bad Request |
| `401` | Unauthorized |
| `403` | Forbidden |
| `404` | Not Found |
| `500` | Internal Server Error |

---

### Authentication

**Basic Auth**: credentials sent as a base64-encoded `Authorization` header:
```
Authorization: Basic YWRtaW46YWRtaW4=
```
Not encrypted, only safe over HTTPS. On failure, server returns `WWW-Authenticate` header.

**Form Auth**: credentials sent in the request body as `x-www-form-urlencoded`:
```
username=tomsmith&password=SuperSecretPassword!
```
Common pattern for web login forms. The server returns a session cookie on success.

**Token Auth**: POST credentials to an `/auth` endpoint, receive a token, pass it in subsequent requests via a `Cookie` header:
```
Cookie: token=abc123xyz
```
Used by Restful Booker for PUT and DELETE operations.

---

### CRUD Operations

**CRUD** = Create, Read, Update, and Delete.tThese are the four fundamental data operations. Maps directly to HTTP methods:

| Operation | Method | Restful Booker Endpoint |
|-----------|--------|------------------------|
| Create | POST | `/booking` |
| Read | GET | `/booking` / `/booking/:id` |
| Update | PUT | `/booking/:id` |
| Delete | DELETE | `/booking/:id` |

---

### Test Scripting

Tests live in the **Scripts → Post-response** tab in Postman. Written in JavaScript using the `pm.*` API.

**Status code assertion:**
```javascript
pm.test("Status is 200", function () {
    pm.response.to.have.status(200);
});
```

**Header assertion:**
```javascript
pm.test("WWW-Authenticate header present", function () {
    pm.expect(pm.response.headers.get("WWW-Authenticate")).to.include("Basic realm");
});
```

**Body assertion:**
```javascript
pm.test("Success message present", function () {
    pm.expect(pm.response.text()).to.include("You logged into a secure area!");
});
```

**Schema validation:**
```javascript
pm.test("Schema is valid", function () {
    const json = pm.response.json();
    pm.expect(json).to.have.property("firstname").that.is.a("string");
    pm.expect(json).to.have.property("totalprice").that.is.a("number");
    pm.expect(json).to.have.property("depositpaid").that.is.a("boolean");
});
```

**Response time assertion:**
```javascript
pm.test("Response time is under 2000ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});
```

---

### Variables & Chaining

Variables eliminate hardcoded values and enable request chaining.

**Types:**

| Type | Scope | Use case |
|------|-------|---------|
| Collection variables | Collection | Runtime-generated values (e.g. `bookingId`, `authToken`) |
| Environment variables | Environment | Config values that change per environment (e.g. `baseUrl`) |

**Setting a variable in a script:**
```javascript
pm.collectionVariables.set("bookingId", pm.response.json().bookingid);
```

**Using a variable in a URL or body:**
```
{{baseUrl}}/booking/{{bookingId}}
```

**Request chaining pattern:**
1. POST Create Booking → captures `bookingId`
2. GET Single Booking → uses `{{bookingId}}`
3. PUT Update Booking → uses `{{bookingId}}` + `{{authToken}}`
4. DELETE Booking → uses `{{bookingId}}` + `{{authToken}}`

---

### Pre-request Scripts

Pre-request scripts run **before** a request fires. Used to set up dynamic values or ensure required variables exist.

**Auto-fetch auth token if missing:**
```javascript
if (!pm.collectionVariables.get("authToken")) {
    pm.sendRequest({
        url: pm.collectionVariables.get("baseUrl") + "/auth",
        method: "POST",
        header: { "Content-Type": "application/json" },
        body: {
            mode: "raw",
            raw: JSON.stringify({ username: "admin", password: "password123" })
        }
    }, function(err, response) {
        pm.collectionVariables.set("authToken", response.json().token);
    });
}
```

---

### Data-Driven Testing

Run the same request multiple times with different data using a CSV file.

**bookings.csv:**
```
firstname,lastname,totalprice,checkin,checkout
Alice,Smith,150,2026-06-01,2026-06-05
Bob,Jones,200,2026-07-10,2026-07-15
Carol,White,99,2026-08-01,2026-08-03
```

**Reference CSV values in request body:**
```json
{
    "firstname": "{{firstname}}",
    "lastname": "{{lastname}}",
    "totalprice": {{totalprice}}
}
```

**Assert against CSV values in test scripts:**
```javascript
pm.expect(json.firstname).to.eql(pm.iterationData.get("firstname"));
```

Newman automatically sets iteration count based on CSV row count. Adding rows scales test cases instantly.

---

### Newman & CI/CD

Newman is Postman's CLI runner. It enables collections to run outside the UI in terminals, build pipelines, or scheduled jobs.

**Basic run:**
```bash
newman run collections/my-collection.json -e environments/dev.json
```

**Data-driven run:**
```bash
newman run collections/my-collection.json \
  -e environments/dev.json \
  --iteration-data data/data.csv
```

Newman exits with a non-zero code on test failures, making it compatible with any CI/CD system (GitHub Actions, Jenkins, GitLab CI, etc.).


## Cheat Sheet

### pm.* API Quick Reference

```javascript
// Response assertions
pm.response.to.have.status(200);
pm.response.to.have.header("Content-Type");
pm.expect(pm.response.responseTime).to.be.below(2000);
pm.expect(pm.response.text()).to.include("some string");
pm.expect(pm.response.json()).to.be.an("array");

// Variables
pm.collectionVariables.set("key", value);
pm.collectionVariables.get("key");
pm.environment.get("key");

// Data-driven
pm.iterationData.get("csvColumnName");

// Sending a request inside a script
pm.sendRequest({ url: "...", method: "GET" }, function(err, res) {
    console.log(res.json());
});
```

### Newman CLI Quick Reference

```bash
# Basic run
newman run collection.json -e environment.json

# Data-driven
newman run collection.json -e environment.json --iteration-data data.csv

# Set iteration count manually
newman run collection.json -n 5

# Export HTML report (requires newman-reporter-html)
newman run collection.json -r html --reporter-html-export report.html
```

### Common QA Testing Patterns

```
Happy path    → valid inputs, expect success response
Sad path      → invalid inputs, expect error response
Auth testing  → valid creds (200), invalid creds (401), no creds (403)
Schema check  → verify field names, types, and structure
Body check    → status code alone is not enough, always check the body
Chaining      → capture IDs/tokens from one response, use in the next
Teardown      → always DELETE test data created during a run
```


## What's Next

Areas to explore to continue building API testing expertise:

### Postman
- **Mock servers**: simulate API endpoints before the backend is built, enabling frontend and test development in parallel
- **Monitor scheduling**: run collections on a schedule (hourly, daily) and get alerted on failures without manual intervention
- **Postman Flows**: visual workflow builder for complex multi-step API logic
- **Auto Documentation**: explore Postman's automated API documentation features, generating web-viewable documentation from collections, including request details, headers, and code snippets, all while updated in real-time. 

### API Testing Depth
- **PATCH requests**: partial updates vs PUT full replacement; understanding when each is appropriate
- **Negative testing**: systematically testing invalid inputs, missing fields, wrong data types, and boundary values
- **Contract testing**: verifying an API's response structure matches what consumers expect, using tools like Pact
- **OAuth 2.0**: industry-standard authorization flow used by most modern APIs (Google, GitHub, etc.)

### Performance & CI/CD
- **Newman HTML reports**: generating visual test reports using `newman-reporter-htmlextra`
- **GitHub Actions integration**: automatically running Newman on every push or pull request
- **k6 or Gatling**: dedicated performance/load testing tools that complement Postman for stress testing

### Broader QA Skills
- **REST Assured**: API testing in Java, common in enterprise QA roles
- **Cypress or Playwright**: Combines UI and API testing with one of my test automation frameworks
- **OpenAPI/Swagger**: reading and testing against API specifications