# ✅ Backend Requirement Specifications – Airbnb Clone

## 1. 🔐 User Authentication

### Objective:

Allow users to securely register, authenticate, and manage sessions using JWT.

### Endpoints:

- `POST /users/register/` – Register a new user
- `POST /users/login/` – Authenticate and obtain access/refresh tokens
- `POST /users/refresh/` – Refresh expired access token
- `GET /users/{user_id}/` – Retrieve user details (authenticated)
- `PATCH /users/{user_id}/` – Update profile (authenticated)

### Input Specifications:

#### Registration (`/users/register/`)

```json
{
  "first_name": "Jane",
  "last_name": "Doe",
  "email": "jane@example.com",
  "password": "strongpassword123"
}
```

#### Login (`/users/login/`)

```json
{
  "email": "jane@example.com",
  "password": "strongpassword123"
}
```

### Output Specifications:

#### Registration (201 Created)

```json
{
  "id": "uuid",
  "email": "jane@example.com",
  "role": "guest",
  "token": {
    "access": "jwt-access-token",
    "refresh": "jwt-refresh-token"
  }
}
```

#### Login (200 OK)

```json
{
  "access": "jwt-access-token",
  "refresh": "jwt-refresh-token"
}
```

### Validation Rules:

- `email`: must be unique and valid format.
- `password`: minimum 8 characters, must include at least one letter and one number.
- `role`: default is `guest` unless promoted.
- Access to update or view user info is limited to the owner or admin.

### Performance Criteria:

- Authentication requests should respond within 500ms.
- Rate limit: max 5 login attempts per minute per IP.

---

## 2. 🏠 Property Management

### Objective:

Enable hosts to create, update, and manage property listings.

### Endpoints:

- `POST /properties/` – Create property (host-only)
- `GET /properties/` – List all properties
- `GET /properties/{property_id}/` – Retrieve specific property details
- `PATCH /properties/{property_id}/` – Update property (host-only)
- `DELETE /properties/{property_id}/` – Remove listing (host-only)

### Input Specifications (POST /properties/)

```json
{
  "name": "Cozy Apartment in Nairobi",
  "description": "A 2-bedroom apartment with a beautiful view",
  "location": "Nairobi, Kenya",
  "price_per_night": 50.0
}
```

### Output Specifications:

#### Success (201 Created)

```json
{
  "id": "uuid",
  "host_id": "user_uuid",
  "name": "Cozy Apartment in Nairobi",
  "location": "Nairobi, Kenya",
  "price_per_night": 50.0,
  "created_at": "2025-06-27T12:00:00Z"
}
```

### Validation Rules:

- `name`, `description`, `location`, and `price_per_night` are required.
- `price_per_night`: must be a positive decimal.
- Only the property’s host can update or delete it.
- Host must be authenticated with role = `host`.

### Performance Criteria:

- List properties: supports pagination (e.g., 20 per page)
- Must respond within 800ms
- Caching via Redis for read-heavy queries

---

## 3. 📅 Booking System

### Objective:

Allow guests to book properties, ensuring availability and accurate pricing.

### Endpoints:

- `POST /bookings/` – Create a new booking
- `GET /bookings/` – List bookings for authenticated user
- `GET /bookings/{booking_id}/` – Retrieve a specific booking
- `PATCH /bookings/{booking_id}/` – Cancel or update a booking (guest-only)

### Input Specifications (POST /bookings/)

```json
{
  "property_id": "property_uuid",
  "start_date": "2025-07-01",
  "end_date": "2025-07-04"
}
```

### Output Specifications:

#### Success (201 Created)

```json
{
  "id": "booking_uuid",
  "user_id": "user_uuid",
  "property_id": "property_uuid",
  "start_date": "2025-07-01",
  "end_date": "2025-07-04",
  "total_price": 150.0,
  "status": "pending"
}
```

### Validation Rules:

- `start_date` must be in the future.
- `end_date` must be after `start_date`.
- System calculates `total_price = price_per_night × number_of_nights`.
- Cannot book overlapping dates for the same property.
- Guests can only book as authenticated users with role = `guest`.

### Performance Criteria:

- Availability check and booking should complete in <1s.
- Write operations trigger invalidation of related property cache.
- High-concurrency safe using transactions or DB-level locks.
