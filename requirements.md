# Airbnb Clone Backend Requirement Specifications

**Repository:** `alx-airbnb-project-documentation`  
**File:** `requirements.md`  
**Objective:** Document the technical and functional requirements for key backend features of the Airbnb Clone project.

---

## 🧩 Overview

This document outlines the backend requirements for the **Airbnb Clone** project.  
It includes **User Authentication**, **Property Management**, and **Booking System** features — detailing their APIs, validation rules, inputs/outputs, and performance expectations.

---

## 1. User Authentication

### 🎯 Objective

Provide secure user account management with registration, login, password reset, and email verification functionalities.

### 🧱 Entity Structure

| Field | Type | Description |
|-------|------|--------------|
| id | UUID | Unique user identifier |
| name | String | Full name of the user |
| email | String | Unique, verified email address |
| password_hash | String | Encrypted password |
| role | Enum | “guest”, “host”, or “admin” |
| verified | Boolean | Email verification status |
| created_at | DateTime | Account creation timestamp |
| updated_at | DateTime | Last update timestamp |

---

### 🔹 API Endpoints

#### **POST `/api/auth/register`**

Registers a new user.

**Request:**

```json
{
  "email": "user@example.com",
  "password": "P@ssw0rd!",
  "name": "Alex Kiprono",
  "role": "guest"
}
Validations:

Email format valid and unique.

Password ≥ 8 chars, strong (uppercase, lowercase, digit, special char).

Name ≤ 100 chars.

Responses:

201 Created – User created.

409 Conflict – Email already in use.

POST /api/auth/login
Authenticates a user and returns JWT tokens.

Request:

json
Copy code
{
  "email": "user@example.com",
  "password": "P@ssw0rd!"
}
Response:

json
Copy code
{
  "access_token": "<jwt>",
  "refresh_token": "<token>",
  "expires_in": 3600,
  "user": { "id": "uuid", "email": "user@example.com", "role": "guest" }
}
Errors:

401 Unauthorized – Invalid credentials.

423 Locked – Too many failed attempts.

POST /api/auth/forgot-password
Sends a password reset link via email.

Request:

json
Copy code
{ "email": "user@example.com" }
Response:
200 OK – Always, even if user not found (to prevent enumeration).

POST /api/auth/reset-password
Resets user password.

Request:

json
Copy code
{
  "token": "reset-token",
  "new_password": "NewP@ssw0rd!"
}
Response:
200 OK – Password updated.

🔐 Security & Performance
Passwords hashed using bcrypt or argon2.

JWT tokens expire in 1 hour; refresh tokens valid for 30 days.

Rate limit: 5 requests/min for register; 10/min for login.

All requests over HTTPS (TLS 1.2+).

95th percentile response time: < 300ms.

2. Property Management
🎯 Objective
Allow hosts to create, update, and manage property listings, including availability, amenities, and photos.

🧱 Entity Structure
Field Type Description
id UUID Unique property ID
host_id UUID Owner (user) ID
title String Listing title
description Text Full description
location Object Address, latitude, longitude
base_price Decimal Price per night
capacity Integer Number of guests allowed
amenities Array List of amenities
is_published Boolean Visibility status
created_at DateTime Creation timestamp

🔹 API Endpoints
POST /api/listings
Creates a new listing.

Request:

json
Copy code
{
  "title": "Modern Apartment",
  "description": "Spacious 2BR apartment",
  "location": { "address": "Nairobi, Kenya", "lat": -1.283, "lng": 36.817 },
  "base_price": 60,
  "currency": "USD",
  "capacity": 4,
  "amenities": ["wifi", "kitchen"]
}
Response:
201 Created – Returns listing ID and details.

GET /api/listings
Search and filter listings.

Query Parameters:
?location=Nairobi&min_price=50&max_price=200&capacity=2&page=1&per_page=20

Response:

json
Copy code
{
  "data": [{ "id": "uuid", "title": "Modern Apartment" }],
  "meta": { "page": 1, "per_page": 20, "total": 50 }
}
PUT /api/listings/{id}
Updates a property.

Request: Same as create.

Response:
200 OK – Updated listing returned.

DELETE /api/listings/{id}
Deletes a property (soft delete).

Response:
204 No Content

📦 Media Upload
POST /api/listings/{id}/photos

Upload images (jpg, png, or webp).

Max 10MB per file.

Stored in S3-compatible storage.

⚙️ Validation Rules
Base price ≥ 0.

Title: 5–150 chars.

Latitude/Longitude valid ranges.

Host must own the listing to update or delete.

🚀 Performance Criteria
Search response time < 300ms.

Listing creation/update < 500ms.

Pagination required (max 100 per page).

Serve media via CDN.

3. Booking System
🎯 Objective
Enable guests to book properties, manage check-in/check-out, and handle payments and cancellations.

🧱 Entity Structure
Field Type Description
id UUID Booking ID
listing_id UUID Related listing
guest_id UUID Guest user ID
checkin_date Date Check-in date
checkout_date Date Check-out date
total_amount Decimal Total booking price
status Enum “pending”, “confirmed”, “canceled”
payment_id UUID Linked payment
created_at DateTime Created timestamp

🔹 API Endpoints
POST /api/bookings
Creates a booking.

Request:

json
Copy code
{
  "listing_id": "uuid",
  "checkin": "2025-12-01",
  "checkout": "2025-12-05",
  "guests": 2,
  "payment_method": { "type": "mpesa", "token": "..." }
}
Steps:

Validate listing availability.

Calculate total amount.

Lock dates to prevent overlap.

Create booking & process payment.

Response:

json
Copy code
{
  "id": "uuid",
  "status": "confirmed",
  "total_amount": 240
}
GET /api/bookings/{id}
Fetches booking details for guest/host/admin.

Response:

json
Copy code
{
  "id": "uuid",
  "listing": "Modern Apartment",
  "status": "confirmed"
}
PUT /api/bookings/{id}/cancel
Cancels an active booking.

Request:

json
Copy code
{ "reason": "Change of plans" }
Response:
200 OK – Booking canceled per policy.

💳 Payments Integration
Supported: Stripe and M-Pesa API.

Payment confirmation triggers booking finalization.

Refunds follow cancellation policy rules.

Webhook handling for payment success/failure.

⚙️ Validation Rules
checkin < checkout

Prevent overlapping bookings (atomic DB check)

Validate guest count ≤ property capacity

🚀 Performance Criteria
Booking creation (incl. DB + validation): < 500ms

Payment confirmation: async, status polled every 10s

Release unconfirmed bookings after 15 mins

4. Cross-Cutting Requirements
Category Requirement
Security All communication via HTTPS, JWT authentication, input validation
Logging Log API errors and performance metrics
Monitoring Track booking rates, errors, and payment failures
Scalability Support 100 concurrent requests/sec per service
Testing Unit, integration, and load tests for major flows
Data Privacy Encrypt PII, support account deletion

5. Deployment Notes
Database: PostgreSQL (for structured data)

Cache: Redis (for sessions and availability locks)

Storage: S3-compatible bucket for photos

Search: Elasticsearch / OpenSearch for listing search

Queue: Celery or BullMQ for async tasks (email, notifications)

CI/CD: GitHub Actions with automated testing and deployment
