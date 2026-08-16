# CleanConnect — Detailed Data Model

## 1. Overview

This document defines the recommended data model for the CleanConnect local cleaning marketplace.

The model is designed to support:

* Customers
* Cleaners
* Cleaning companies
* Location-based discovery
* Services
* Pricing
* Availability
* Bookings
* Recurring bookings
* Payments
* Cleaner payouts
* Messaging
* Ratings and reviews
* Verification
* Notifications
* Disputes
* Promotions
* Administration
* Audit logging

The recommended primary database is **PostgreSQL** because the application requires strong relational integrity, transactions, geographic queries, and financial consistency.

---

# 2. High-Level Entity Relationship

```text
                              ┌──────────────────┐
                              │      USERS       │
                              └────────┬─────────┘
                                       │
                     ┌─────────────────┼─────────────────┐
                     │                 │                 │
                     ▼                 ▼                 ▼
              ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
              │  CUSTOMERS  │   │  CLEANERS   │   │   ADMINS    │
              └──────┬──────┘   └──────┬──────┘   └─────────────┘
                     │                 │
                     │                 ├──────────────┐
                     │                 │              │
                     ▼                 ▼              ▼
              ┌─────────────┐   ┌─────────────┐ ┌──────────────┐
              │  ADDRESSES  │   │  SERVICES   │ │ VERIFICATION │
              └──────┬──────┘   └──────┬──────┘ └──────────────┘
                     │                 │
                     │                 │
                     └────────┬────────┘
                              ▼
                       ┌─────────────┐
                       │  BOOKINGS   │
                       └──────┬──────┘
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
       ┌───────────┐   ┌─────────────┐   ┌─────────────┐
       │ PAYMENTS  │   │   REVIEWS   │   │  MESSAGES   │
       └─────┬─────┘   └─────────────┘   └─────────────┘
             │
             ▼
       ┌─────────────┐
       │   PAYOUTS   │
       └─────────────┘
```

---

# 3. Database Conventions

## Primary Keys

Use UUIDs for application-level IDs.

Example:

```text
550e8400-e29b-41d4-a716-446655440000
```

Recommended:

```sql
id UUID PRIMARY KEY
```

## Timestamps

All tables should use:

```sql
created_at TIMESTAMPTZ NOT NULL
updated_at TIMESTAMPTZ NOT NULL
```

Store timestamps in UTC.

Convert timestamps to the user's local timezone in the application.

## Soft Deletion

Entities that should not be permanently deleted should use:

```sql
deleted_at TIMESTAMPTZ NULL
```

Examples:

* Users
* Cleaners
* Services
* Addresses
* Reviews

---

# 4. USERS

The `users` table represents every authenticated person using the platform.

## Table: `users`

| Field             | Type         | Required | Description                         |
| ----------------- | ------------ | -------: | ----------------------------------- |
| id                | UUID         |      Yes | Primary key                         |
| email             | VARCHAR(255) |      Yes | User email                          |
| phone             | VARCHAR(30)  |       No | Phone number                        |
| password_hash     | TEXT         |       No | Password hash                       |
| first_name        | VARCHAR(100) |      Yes | First name                          |
| last_name         | VARCHAR(100) |      Yes | Last name                           |
| profile_photo_url | TEXT         |       No | Profile photo                       |
| role              | ENUM         |      Yes | CUSTOMER, CLEANER, ADMIN            |
| status            | ENUM         |      Yes | ACTIVE, SUSPENDED, PENDING, DELETED |
| email_verified_at | TIMESTAMPTZ  |       No | Email verification date             |
| phone_verified_at | TIMESTAMPTZ  |       No | Phone verification date             |
| last_login_at     | TIMESTAMPTZ  |       No | Last login                          |
| created_at        | TIMESTAMPTZ  |      Yes | Creation date                       |
| updated_at        | TIMESTAMPTZ  |      Yes | Last update                         |
| deleted_at        | TIMESTAMPTZ  |       No | Soft deletion                       |

### Constraints

```text
email must be unique
phone should be unique when provided
```

---

# 5. CUSTOMER PROFILES

Customer-specific information should be stored separately from authentication information.

## Table: `customer_profiles`

| Field              | Type        | Required | Description             |
| ------------------ | ----------- | -------: | ----------------------- |
| id                 | UUID        |      Yes | Primary key             |
| user_id            | UUID        |      Yes | FK → users.id           |
| preferred_language | VARCHAR(10) |       No | Preferred language      |
| preferred_currency | VARCHAR(3)  |      Yes | Currency                |
| marketing_opt_in   | BOOLEAN     |      Yes | Marketing permission    |
| notes              | TEXT        |       No | Internal/customer notes |
| created_at         | TIMESTAMPTZ |      Yes | Creation date           |
| updated_at         | TIMESTAMPTZ |      Yes | Last update             |

### Relationship

```text
users 1 ─── 1 customer_profiles
```

---

# 6. CLEANER PROFILES

The cleaner profile contains marketplace information.

## Table: `cleaner_profiles`

| Field                  | Type          | Required | Description                 |
| ---------------------- | ------------- | -------: | --------------------------- |
| id                     | UUID          |      Yes | Primary key                 |
| user_id                | UUID          |      Yes | FK → users.id               |
| business_name          | VARCHAR(255)  |       No | Business name               |
| bio                    | TEXT          |       No | Cleaner description         |
| years_experience       | INTEGER       |       No | Experience                  |
| hourly_rate            | NUMERIC(10,2) |       No | Base hourly rate            |
| minimum_booking_amount | NUMERIC(10,2) |       No | Minimum booking             |
| currency               | VARCHAR(3)    |      Yes | Currency                    |
| rating_average         | NUMERIC(3,2)  |      Yes | Average rating              |
| review_count           | INTEGER       |      Yes | Number of reviews           |
| completed_jobs_count   | INTEGER       |      Yes | Completed jobs              |
| verification_status    | ENUM          |      Yes | PENDING, VERIFIED, REJECTED |
| is_featured            | BOOLEAN       |      Yes | Featured cleaner            |
| is_accepting_bookings  | BOOLEAN       |      Yes | Currently accepting jobs    |
| created_at             | TIMESTAMPTZ   |      Yes | Creation date               |
| updated_at             | TIMESTAMPTZ   |      Yes | Last update                 |
| deleted_at             | TIMESTAMPTZ   |       No | Soft deletion               |

### Relationship

```text
users 1 ─── 1 cleaner_profiles
```

---

# 7. CLEANING COMPANIES

The system should support both individual cleaners and companies.

## Table: `cleaning_companies`

| Field               | Type         | Required | Description                |
| ------------------- | ------------ | -------: | -------------------------- |
| id                  | UUID         |      Yes | Primary key                |
| owner_user_id       | UUID         |      Yes | Company owner              |
| name                | VARCHAR(255) |      Yes | Company name               |
| description         | TEXT         |       No | Company description        |
| logo_url            | TEXT         |       No | Company logo               |
| phone               | VARCHAR(30)  |       No | Business phone             |
| email               | VARCHAR(255) |       No | Business email             |
| website_url         | TEXT         |       No | Website                    |
| registration_number | VARCHAR(100) |       No | Business registration      |
| status              | ENUM         |      Yes | ACTIVE, SUSPENDED, PENDING |
| created_at          | TIMESTAMPTZ  |      Yes | Creation date              |
| updated_at          | TIMESTAMPTZ  |      Yes | Last update                |

---

# 8. CLEANER COMPANY MEMBERS

A company may have multiple cleaners.

## Table: `company_members`

| Field      | Type        | Required | Description                |
| ---------- | ----------- | -------: | -------------------------- |
| id         | UUID        |      Yes | Primary key                |
| company_id | UUID        |      Yes | FK → cleaning_companies.id |
| cleaner_id | UUID        |      Yes | FK → cleaner_profiles.id   |
| role       | ENUM        |      Yes | OWNER, MANAGER, CLEANER    |
| status     | ENUM        |      Yes | ACTIVE, INVITED, REMOVED   |
| joined_at  | TIMESTAMPTZ |      Yes | Join date                  |

---

# 9. ADDRESSES

Addresses should be reusable for customers and bookings.

## Table: `addresses`

| Field               | Type          | Required | Description        |
| ------------------- | ------------- | -------: | ------------------ |
| id                  | UUID          |      Yes | Primary key        |
| user_id             | UUID          |       No | Address owner      |
| label               | VARCHAR(100)  |       No | Home, Office, etc. |
| address_line_1      | VARCHAR(255)  |      Yes | Address            |
| address_line_2      | VARCHAR(255)  |       No | Apartment/unit     |
| city                | VARCHAR(100)  |      Yes | City               |
| state               | VARCHAR(100)  |       No | State/province     |
| postal_code         | VARCHAR(30)   |       No | Postal code        |
| country_code        | CHAR(2)       |      Yes | ISO country        |
| latitude            | DECIMAL(10,7) |       No | Latitude           |
| longitude           | DECIMAL(10,7) |       No | Longitude          |
| access_instructions | TEXT          |       No | Entry instructions |
| is_default          | BOOLEAN       |      Yes | Default address    |
| created_at          | TIMESTAMPTZ   |      Yes | Creation date      |
| updated_at          | TIMESTAMPTZ   |      Yes | Last update        |

### Important

Use a geospatial extension such as **PostGIS** for location-based searches.

---

# 10. CLEANER SERVICE AREAS

Cleaners should define where they operate.

## Table: `cleaner_service_areas`

| Field            | Type          | Required | Description      |
| ---------------- | ------------- | -------: | ---------------- |
| id               | UUID          |      Yes | Primary key      |
| cleaner_id       | UUID          |      Yes | FK               |
| name             | VARCHAR(255)  |      Yes | Area name        |
| center_latitude  | DECIMAL(10,7) |      Yes | Center latitude  |
| center_longitude | DECIMAL(10,7) |      Yes | Center longitude |
| radius_km        | DECIMAL(8,2)  |      Yes | Service radius   |
| created_at       | TIMESTAMPTZ   |      Yes | Creation date    |

For advanced implementations, use a PostGIS `geography` or `geometry` field.

---

# 11. SERVICE CATEGORIES

## Table: `service_categories`

| Field       | Type         | Required | Description   |
| ----------- | ------------ | -------: | ------------- |
| id          | UUID         |      Yes | Primary key   |
| name        | VARCHAR(150) |      Yes | Category name |
| description | TEXT         |       No | Description   |
| icon_url    | TEXT         |       No | Category icon |
| sort_order  | INTEGER      |      Yes | Display order |
| is_active   | BOOLEAN      |      Yes | Active status |
| created_at  | TIMESTAMPTZ  |      Yes | Creation date |
| updated_at  | TIMESTAMPTZ  |      Yes | Last update   |

Examples:

```text
Standard Cleaning
Deep Cleaning
Move-In Cleaning
Move-Out Cleaning
Airbnb Cleaning
```

---

# 12. SERVICES

A service represents a cleaning service offered by the marketplace.

## Table: `services`

| Field            | Type          | Required | Description           |
| ---------------- | ------------- | -------: | --------------------- |
| id               | UUID          |      Yes | Primary key           |
| category_id      | UUID          |      Yes | FK                    |
| name             | VARCHAR(150)  |      Yes | Service name          |
| description      | TEXT          |       No | Description           |
| pricing_type     | ENUM          |      Yes | FIXED, HOURLY, CUSTOM |
| base_price       | NUMERIC(10,2) |       No | Base price            |
| currency         | VARCHAR(3)    |      Yes | Currency              |
| duration_minutes | INTEGER       |       No | Estimated duration    |
| is_active        | BOOLEAN       |      Yes | Active status         |
| created_at       | TIMESTAMPTZ   |      Yes | Creation date         |
| updated_at       | TIMESTAMPTZ   |      Yes | Last update           |

---

# 13. CLEANER SERVICES

This connects cleaners to the services they provide.

## Table: `cleaner_services`

| Field            | Type          | Required | Description   |
| ---------------- | ------------- | -------: | ------------- |
| id               | UUID          |      Yes | Primary key   |
| cleaner_id       | UUID          |      Yes | FK            |
| service_id       | UUID          |      Yes | FK            |
| price            | NUMERIC(10,2) |      Yes | Cleaner price |
| minimum_price    | NUMERIC(10,2) |       No | Minimum price |
| duration_minutes | INTEGER       |       No | Duration      |
| is_active        | BOOLEAN       |      Yes | Active status |
| created_at       | TIMESTAMPTZ   |      Yes | Creation date |
| updated_at       | TIMESTAMPTZ   |      Yes | Last update   |

---

# 14. SERVICE ADD-ONS

Additional services can be attached to a booking.

## Table: `service_addons`

| Field            | Type          | Required | Description     |
| ---------------- | ------------- | -------: | --------------- |
| id               | UUID          |      Yes | Primary key     |
| name             | VARCHAR(150)  |      Yes | Add-on name     |
| description      | TEXT          |       No | Description     |
| price            | NUMERIC(10,2) |      Yes | Price           |
| duration_minutes | INTEGER       |       No | Additional time |
| is_active        | BOOLEAN       |      Yes | Active          |
| created_at       | TIMESTAMPTZ   |      Yes | Creation date   |

Examples:

```text
Oven Cleaning
Refrigerator Cleaning
Laundry
Interior Windows
Ironing
```

---

# 15. CLEANER AVAILABILITY

## Table: `cleaner_availability`

Stores recurring availability.

| Field       | Type         | Required | Description   |
| ----------- | ------------ | -------: | ------------- |
| id          | UUID         |      Yes | Primary key   |
| cleaner_id  | UUID         |      Yes | FK            |
| day_of_week | INTEGER      |      Yes | 0–6           |
| start_time  | TIME         |      Yes | Start time    |
| end_time    | TIME         |      Yes | End time      |
| timezone    | VARCHAR(100) |      Yes | Timezone      |
| is_active   | BOOLEAN      |      Yes | Active        |
| created_at  | TIMESTAMPTZ  |      Yes | Creation date |
| updated_at  | TIMESTAMPTZ  |      Yes | Last update   |

---

# 16. CLEANER TIME OFF

## Table: `cleaner_time_off`

Stores unavailable periods.

| Field      | Type         | Required | Description   |
| ---------- | ------------ | -------: | ------------- |
| id         | UUID         |      Yes | Primary key   |
| cleaner_id | UUID         |      Yes | FK            |
| start_at   | TIMESTAMPTZ  |      Yes | Start         |
| end_at     | TIMESTAMPTZ  |      Yes | End           |
| reason     | VARCHAR(255) |       No | Reason        |
| created_at | TIMESTAMPTZ  |      Yes | Creation date |

---

# 17. BOOKINGS

Bookings are the central transactional entity.

## Table: `bookings`

| Field                | Type          | Required | Description               |
| -------------------- | ------------- | -------: | ------------------------- |
| id                   | UUID          |      Yes | Primary key               |
| booking_number       | VARCHAR(30)   |      Yes | Human-readable booking ID |
| customer_id          | UUID          |      Yes | FK → users.id             |
| cleaner_id           | UUID          |      Yes | FK → cleaner_profiles.id  |
| address_id           | UUID          |      Yes | FK → addresses.id         |
| service_id           | UUID          |      Yes | FK → services.id          |
| status               | ENUM          |      Yes | Booking status            |
| scheduled_start_at   | TIMESTAMPTZ   |      Yes | Start                     |
| scheduled_end_at     | TIMESTAMPTZ   |      Yes | End                       |
| timezone             | VARCHAR(100)  |      Yes | Local timezone            |
| bedrooms             | INTEGER       |       No | Number bedrooms           |
| bathrooms            | DECIMAL(3,1)  |       No | Number bathrooms          |
| property_size        | INTEGER       |       No | Property size             |
| property_size_unit   | ENUM          |       No | SQFT, SQM                 |
| special_instructions | TEXT          |       No | Customer instructions     |
| subtotal             | NUMERIC(10,2) |      Yes | Subtotal                  |
| discount_amount      | NUMERIC(10,2) |      Yes | Discount                  |
| service_fee          | NUMERIC(10,2) |      Yes | Platform fee              |
| tax_amount           | NUMERIC(10,2) |      Yes | Tax                       |
| total_amount         | NUMERIC(10,2) |      Yes | Customer total            |
| cleaner_earnings     | NUMERIC(10,2) |      Yes | Cleaner earnings          |
| currency             | VARCHAR(3)    |      Yes | Currency                  |
| cancellation_reason  | TEXT          |       No | Cancellation reason       |
| created_at           | TIMESTAMPTZ   |      Yes | Created                   |
| updated_at           | TIMESTAMPTZ   |      Yes | Updated                   |
| cancelled_at         | TIMESTAMPTZ   |       No | Cancellation time         |
| completed_at         | TIMESTAMPTZ   |       No | Completion time           |

---

# 18. BOOKING ITEMS

A booking may contain multiple services/add-ons.

## Table: `booking_items`

| Field       | Type          | Required | Description          |
| ----------- | ------------- | -------: | -------------------- |
| id          | UUID          |      Yes | Primary key          |
| booking_id  | UUID          |      Yes | FK                   |
| service_id  | UUID          |       No | Service              |
| addon_id    | UUID          |       No | Add-on               |
| description | VARCHAR(255)  |      Yes | Snapshot description |
| quantity    | INTEGER       |      Yes | Quantity             |
| unit_price  | NUMERIC(10,2) |      Yes | Price                |
| total_price | NUMERIC(10,2) |      Yes | Total                |
| created_at  | TIMESTAMPTZ   |      Yes | Creation date        |

Prices should be stored as snapshots so historical bookings do not change when a cleaner later changes their pricing.

---

# 19. BOOKING STATUS HISTORY

Every important status change should be recorded.

## Table: `booking_status_history`

| Field              | Type        | Required | Description            |
| ------------------ | ----------- | -------: | ---------------------- |
| id                 | UUID        |      Yes | Primary key            |
| booking_id         | UUID        |      Yes | FK                     |
| old_status         | ENUM        |       No | Previous status        |
| new_status         | ENUM        |      Yes | New status             |
| changed_by_user_id | UUID        |       No | User making change     |
| notes              | TEXT        |       No | Additional information |
| created_at         | TIMESTAMPTZ |      Yes | Change time            |

---

# 20. RECURRING BOOKINGS

## Table: `recurring_bookings`

| Field             | Type        | Required | Description                       |
| ----------------- | ----------- | -------: | --------------------------------- |
| id                | UUID        |      Yes | Primary key                       |
| customer_id       | UUID        |      Yes | FK                                |
| cleaner_id        | UUID        |      Yes | FK                                |
| service_id        | UUID        |      Yes | FK                                |
| address_id        | UUID        |      Yes | FK                                |
| frequency         | ENUM        |      Yes | WEEKLY, BIWEEKLY, MONTHLY, CUSTOM |
| day_of_week       | INTEGER     |       No | Preferred weekday                 |
| preferred_time    | TIME        |       No | Preferred time                    |
| start_date        | DATE        |      Yes | Start                             |
| end_date          | DATE        |       No | End                               |
| status            | ENUM        |      Yes | ACTIVE, PAUSED, CANCELLED         |
| next_booking_date | DATE        |       No | Next occurrence                   |
| created_at        | TIMESTAMPTZ |      Yes | Created                           |
| updated_at        | TIMESTAMPTZ |      Yes | Updated                           |

Generated bookings should reference the recurring booking.

---

# 21. PAYMENTS

Payments should be kept separate from bookings because a booking may have multiple payment attempts.

## Table: `payments`

| Field               | Type          | Required | Description                                 |
| ------------------- | ------------- | -------: | ------------------------------------------- |
| id                  | UUID          |      Yes | Primary key                                 |
| booking_id          | UUID          |      Yes | FK                                          |
| customer_id         | UUID          |      Yes | FK                                          |
| payment_provider    | VARCHAR(50)   |      Yes | Stripe/etc.                                 |
| provider_payment_id | VARCHAR(255)  |      Yes | Provider ID                                 |
| amount              | NUMERIC(10,2) |      Yes | Amount                                      |
| currency            | VARCHAR(3)    |      Yes | Currency                                    |
| status              | ENUM          |      Yes | PENDING, AUTHORIZED, PAID, FAILED, REFUNDED |
| payment_method_type | VARCHAR(50)   |       No | Card, wallet, etc.                          |
| failure_reason      | TEXT          |       No | Failure                                     |
| paid_at             | TIMESTAMPTZ   |       No | Payment time                                |
| created_at          | TIMESTAMPTZ   |      Yes | Created                                     |
| updated_at          | TIMESTAMPTZ   |      Yes | Updated                                     |

---

# 22. REFUNDS

## Table: `refunds`

| Field              | Type          | Required | Description                |
| ------------------ | ------------- | -------: | -------------------------- |
| id                 | UUID          |      Yes | Primary key                |
| payment_id         | UUID          |      Yes | FK                         |
| booking_id         | UUID          |      Yes | FK                         |
| amount             | NUMERIC(10,2) |      Yes | Refund amount              |
| reason             | TEXT          |      Yes | Refund reason              |
| provider_refund_id | VARCHAR(255)  |       No | Payment provider ID        |
| status             | ENUM          |      Yes | PENDING, COMPLETED, FAILED |
| created_at         | TIMESTAMPTZ   |      Yes | Created                    |
| completed_at       | TIMESTAMPTZ   |       No | Completed                  |

---

# 23. CLEANER PAYOUTS

## Table: `payouts`

| Field              | Type          | Required | Description                       |
| ------------------ | ------------- | -------: | --------------------------------- |
| id                 | UUID          |      Yes | Primary key                       |
| cleaner_id         | UUID          |      Yes | FK                                |
| booking_id         | UUID          |       No | Related booking                   |
| amount             | NUMERIC(10,2) |      Yes | Payout amount                     |
| currency           | VARCHAR(3)    |      Yes | Currency                          |
| provider           | VARCHAR(50)   |      Yes | Payout provider                   |
| provider_payout_id | VARCHAR(255)  |       No | Provider ID                       |
| status             | ENUM          |      Yes | PENDING, PROCESSING, PAID, FAILED |
| scheduled_at       | TIMESTAMPTZ   |       No | Scheduled                         |
| paid_at            | TIMESTAMPTZ   |       No | Paid                              |
| created_at         | TIMESTAMPTZ   |      Yes | Created                           |

---

# 24. CLEANER PAYMENT ACCOUNTS

## Table: `cleaner_payment_accounts`

Stores references to the cleaner's payment provider account.

Do **not** store bank account numbers or card information directly unless absolutely necessary and compliant with applicable financial/security requirements.

| Field               | Type         | Required | Description                 |
| ------------------- | ------------ | -------: | --------------------------- |
| id                  | UUID         |      Yes | Primary key                 |
| cleaner_id          | UUID         |      Yes | FK                          |
| provider            | VARCHAR(50)  |      Yes | Payment provider            |
| provider_account_id | VARCHAR(255) |      Yes | External account ID         |
| status              | ENUM         |      Yes | PENDING, ACTIVE, RESTRICTED |
| created_at          | TIMESTAMPTZ  |      Yes | Created                     |
| updated_at          | TIMESTAMPTZ  |      Yes | Updated                     |

---

# 25. REVIEWS

## Table: `reviews`

| Field                  | Type        | Required | Description                |
| ---------------------- | ----------- | -------: | -------------------------- |
| id                     | UUID        |      Yes | Primary key                |
| booking_id             | UUID        |      Yes | FK                         |
| customer_id            | UUID        |      Yes | FK                         |
| cleaner_id             | UUID        |      Yes | FK                         |
| overall_rating         | INTEGER     |      Yes | 1–5                        |
| quality_rating         | INTEGER     |       No | 1–5                        |
| professionalism_rating | INTEGER     |       No | 1–5                        |
| punctuality_rating     | INTEGER     |       No | 1–5                        |
| communication_rating   | INTEGER     |       No | 1–5                        |
| comment                | TEXT        |       No | Written review             |
| status                 | ENUM        |      Yes | PUBLISHED, HIDDEN, FLAGGED |
| created_at             | TIMESTAMPTZ |      Yes | Created                    |
| updated_at             | TIMESTAMPTZ |      Yes | Updated                    |

### Constraint

A customer should normally be allowed only one review per completed booking.

```text
UNIQUE(booking_id, customer_id)
```

---

# 26. REVIEW RESPONSES

## Table: `review_responses`

| Field      | Type        | Required | Description |
| ---------- | ----------- | -------: | ----------- |
| id         | UUID        |      Yes | Primary key |
| review_id  | UUID        |      Yes | FK          |
| cleaner_id | UUID        |      Yes | FK          |
| response   | TEXT        |      Yes | Response    |
| created_at | TIMESTAMPTZ |      Yes | Created     |
| updated_at | TIMESTAMPTZ |      Yes | Updated     |

---

# 27. FAVORITE CLEANERS

## Table: `customer_favorite_cleaners`

| Field       | Type        | Required | Description |
| ----------- | ----------- | -------: | ----------- |
| id          | UUID        |      Yes | Primary key |
| customer_id | UUID        |      Yes | FK          |
| cleaner_id  | UUID        |      Yes | FK          |
| created_at  | TIMESTAMPTZ |      Yes | Created     |

### Constraint

```text
UNIQUE(customer_id, cleaner_id)
```

---

# 28. MESSAGES

## Table: `conversations`

| Field      | Type        | Required | Description     |
| ---------- | ----------- | -------: | --------------- |
| id         | UUID        |      Yes | Primary key     |
| booking_id | UUID        |       No | Related booking |
| created_at | TIMESTAMPTZ |      Yes | Created         |
| updated_at | TIMESTAMPTZ |      Yes | Updated         |

## Table: `conversation_participants`

| Field           | Type        | Required | Description |
| --------------- | ----------- | -------: | ----------- |
| id              | UUID        |      Yes | Primary key |
| conversation_id | UUID        |      Yes | FK          |
| user_id         | UUID        |      Yes | FK          |
| joined_at       | TIMESTAMPTZ |      Yes | Joined      |

## Table: `messages`

| Field           | Type        | Required | Description         |
| --------------- | ----------- | -------: | ------------------- |
| id              | UUID        |      Yes | Primary key         |
| conversation_id | UUID        |      Yes | FK                  |
| sender_id       | UUID        |      Yes | FK                  |
| message_type    | ENUM        |      Yes | TEXT, IMAGE, SYSTEM |
| body            | TEXT        |       No | Message             |
| attachment_url  | TEXT        |       No | Attachment          |
| read_at         | TIMESTAMPTZ |       No | Read time           |
| created_at      | TIMESTAMPTZ |      Yes | Created             |

---

# 29. NOTIFICATIONS

## Table: `notifications`

| Field      | Type         | Required | Description         |
| ---------- | ------------ | -------: | ------------------- |
| id         | UUID         |      Yes | Primary key         |
| user_id    | UUID         |      Yes | Recipient           |
| type       | VARCHAR(100) |      Yes | Notification type   |
| title      | VARCHAR(255) |      Yes | Title               |
| body       | TEXT         |      Yes | Message             |
| data       | JSONB        |       No | Additional metadata |
| read_at    | TIMESTAMPTZ  |       No | Read time           |
| created_at | TIMESTAMPTZ  |      Yes | Created             |

---

# 30. PUSH DEVICES

## Table: `user_devices`

| Field        | Type         | Required | Description       |
| ------------ | ------------ | -------: | ----------------- |
| id           | UUID         |      Yes | Primary key       |
| user_id      | UUID         |      Yes | FK                |
| platform     | ENUM         |      Yes | IOS, ANDROID, WEB |
| push_token   | TEXT         |      Yes | Push token        |
| device_name  | VARCHAR(255) |       No | Device            |
| last_seen_at | TIMESTAMPTZ  |       No | Last activity     |
| created_at   | TIMESTAMPTZ  |      Yes | Created           |

---

# 31. CLEANER VERIFICATION

## Table: `cleaner_verifications`

| Field              | Type         | Required | Description                                                   |
| ------------------ | ------------ | -------: | ------------------------------------------------------------- |
| id                 | UUID         |      Yes | Primary key                                                   |
| cleaner_id         | UUID         |      Yes | FK                                                            |
| verification_type  | ENUM         |      Yes | IDENTITY, PHONE, EMAIL, BACKGROUND_CHECK, INSURANCE, BUSINESS |
| status             | ENUM         |      Yes | PENDING, APPROVED, REJECTED, EXPIRED                          |
| provider           | VARCHAR(100) |       No | Verification provider                                         |
| external_reference | VARCHAR(255) |       No | External ID                                                   |
| submitted_at       | TIMESTAMPTZ  |      Yes | Submitted                                                     |
| verified_at        | TIMESTAMPTZ  |       No | Verified                                                      |
| expires_at         | TIMESTAMPTZ  |       No | Expiration                                                    |
| rejection_reason   | TEXT         |       No | Rejection                                                     |
| created_at         | TIMESTAMPTZ  |      Yes | Created                                                       |
| updated_at         | TIMESTAMPTZ  |      Yes | Updated                                                       |

---

# 32. DOCUMENTS

If verification documents need to be uploaded, store the files in secure object storage and store only metadata in PostgreSQL.

## Table: `verification_documents`

| Field             | Type         | Required | Description              |
| ----------------- | ------------ | -------: | ------------------------ |
| id                | UUID         |      Yes | Primary key              |
| verification_id   | UUID         |      Yes | FK                       |
| document_type     | VARCHAR(100) |      Yes | ID, insurance, etc.      |
| storage_key       | TEXT         |      Yes | Secure storage reference |
| original_filename | VARCHAR(255) |       No | Original filename        |
| mime_type         | VARCHAR(100) |       No | File type                |
| uploaded_at       | TIMESTAMPTZ  |      Yes | Upload time              |

---

# 33. DISPUTES

## Table: `disputes`

| Field               | Type        | Required | Description                              |
| ------------------- | ----------- | -------: | ---------------------------------------- |
| id                  | UUID        |      Yes | Primary key                              |
| booking_id          | UUID        |      Yes | FK                                       |
| opened_by_user_id   | UUID        |      Yes | User                                     |
| type                | ENUM        |      Yes | QUALITY, PAYMENT, NO_SHOW, DAMAGE, OTHER |
| status              | ENUM        |      Yes | OPEN, INVESTIGATING, RESOLVED, CLOSED    |
| description         | TEXT        |      Yes | Dispute details                          |
| resolution          | TEXT        |       No | Resolution                               |
| resolved_by_user_id | UUID        |       No | Admin                                    |
| resolved_at         | TIMESTAMPTZ |       No | Resolution date                          |
| created_at          | TIMESTAMPTZ |      Yes | Created                                  |
| updated_at          | TIMESTAMPTZ |      Yes | Updated                                  |

---

# 34. DISPUTE EVIDENCE

## Table: `dispute_evidence`

| Field                | Type        | Required | Description                  |
| -------------------- | ----------- | -------: | ---------------------------- |
| id                   | UUID        |      Yes | Primary key                  |
| dispute_id           | UUID        |      Yes | FK                           |
| submitted_by_user_id | UUID        |      Yes | User                         |
| type                 | ENUM        |      Yes | IMAGE, VIDEO, DOCUMENT, TEXT |
| storage_key          | TEXT        |       No | File reference               |
| description          | TEXT        |       No | Evidence description         |
| created_at           | TIMESTAMPTZ |      Yes | Created                      |

---

# 35. PROMOTIONS

## Table: `promotions`

| Field                  | Type          | Required | Description       |
| ---------------------- | ------------- | -------: | ----------------- |
| id                     | UUID          |      Yes | Primary key       |
| code                   | VARCHAR(50)   |      Yes | Promo code        |
| description            | TEXT          |       No | Description       |
| discount_type          | ENUM          |      Yes | PERCENTAGE, FIXED |
| discount_value         | NUMERIC(10,2) |      Yes | Discount          |
| minimum_booking_amount | NUMERIC(10,2) |       No | Minimum           |
| maximum_discount       | NUMERIC(10,2) |       No | Maximum discount  |
| usage_limit            | INTEGER       |       No | Total usage limit |
| per_user_limit         | INTEGER       |       No | Per-user limit    |
| starts_at              | TIMESTAMPTZ   |      Yes | Start             |
| expires_at             | TIMESTAMPTZ   |      Yes | Expiration        |
| is_active              | BOOLEAN       |      Yes | Active            |
| created_at             | TIMESTAMPTZ   |      Yes | Created           |
| updated_at             | TIMESTAMPTZ   |      Yes | Updated           |

---

# 36. PROMOTION USAGE

## Table: `promotion_usage`

| Field           | Type          | Required | Description      |
| --------------- | ------------- | -------: | ---------------- |
| id              | UUID          |      Yes | Primary key      |
| promotion_id    | UUID          |      Yes | FK               |
| booking_id      | UUID          |      Yes | FK               |
| customer_id     | UUID          |      Yes | FK               |
| discount_amount | NUMERIC(10,2) |      Yes | Applied discount |
| created_at      | TIMESTAMPTZ   |      Yes | Created          |

---

# 37. PLATFORM FEES

The platform should have configurable fee rules.

## Table: `fee_rules`

| Field        | Type          | Required | Description                |
| ------------ | ------------- | -------: | -------------------------- |
| id           | UUID          |      Yes | Primary key                |
| name         | VARCHAR(150)  |      Yes | Fee name                   |
| fee_type     | ENUM          |      Yes | PERCENTAGE, FIXED          |
| amount       | NUMERIC(10,4) |      Yes | Fee value                  |
| applies_to   | ENUM          |      Yes | CUSTOMER, CLEANER, BOOKING |
| country_code | CHAR(2)       |       No | Optional country           |
| starts_at    | TIMESTAMPTZ   |      Yes | Start                      |
| ends_at      | TIMESTAMPTZ   |       No | End                        |
| is_active    | BOOLEAN       |      Yes | Active                     |

---

# 38. BOOKING FINANCIAL BREAKDOWN

For financial transparency, store the actual amounts applied to each booking.

## Table: `booking_financials`

| Field                | Type          | Required | Description          |
| -------------------- | ------------- | -------: | -------------------- |
| id                   | UUID          |      Yes | Primary key          |
| booking_id           | UUID          |      Yes | FK                   |
| service_amount       | NUMERIC(10,2) |      Yes | Services             |
| addon_amount         | NUMERIC(10,2) |      Yes | Add-ons              |
| discount_amount      | NUMERIC(10,2) |      Yes | Discounts            |
| customer_fee         | NUMERIC(10,2) |      Yes | Customer fee         |
| cleaner_fee          | NUMERIC(10,2) |      Yes | Cleaner/platform fee |
| tax_amount           | NUMERIC(10,2) |      Yes | Tax                  |
| total_customer_paid  | NUMERIC(10,2) |      Yes | Customer total       |
| total_cleaner_earned | NUMERIC(10,2) |      Yes | Cleaner earnings     |
| platform_revenue     | NUMERIC(10,2) |      Yes | Platform revenue     |
| currency             | VARCHAR(3)    |      Yes | Currency             |
| created_at           | TIMESTAMPTZ   |      Yes | Created              |

---

# 39. AUDIT LOG

Administrative and security-sensitive actions should be logged.

## Table: `audit_logs`

| Field         | Type         | Required | Description            |
| ------------- | ------------ | -------: | ---------------------- |
| id            | UUID         |      Yes | Primary key            |
| actor_user_id | UUID         |       No | User performing action |
| action        | VARCHAR(100) |      Yes | Action                 |
| entity_type   | VARCHAR(100) |      Yes | Entity                 |
| entity_id     | UUID         |       No | Entity ID              |
| old_values    | JSONB        |       No | Previous values        |
| new_values    | JSONB        |       No | New values             |
| ip_address    | INET         |       No | IP address             |
| user_agent    | TEXT         |       No | Browser/device         |
| created_at    | TIMESTAMPTZ  |      Yes | Created                |

---

# 40. SUPPORT TICKETS

## Table: `support_tickets`

| Field               | Type         | Required | Description                         |
| ------------------- | ------------ | -------: | ----------------------------------- |
| id                  | UUID         |      Yes | Primary key                         |
| user_id             | UUID         |      Yes | User                                |
| booking_id          | UUID         |       No | Related booking                     |
| subject             | VARCHAR(255) |      Yes | Subject                             |
| description         | TEXT         |      Yes | Issue                               |
| priority            | ENUM         |      Yes | LOW, MEDIUM, HIGH, URGENT           |
| status              | ENUM         |      Yes | OPEN, IN_PROGRESS, RESOLVED, CLOSED |
| assigned_to_user_id | UUID         |       No | Support agent                       |
| created_at          | TIMESTAMPTZ  |      Yes | Created                             |
| updated_at          | TIMESTAMPTZ  |      Yes | Updated                             |
| resolved_at         | TIMESTAMPTZ  |       No | Resolved                            |

---

# 41. DATABASE RELATIONSHIPS

The most important relationships are:

```text
users
  │
  ├── customer_profiles
  │
  ├── cleaner_profiles
  │      │
  │      ├── cleaner_services
  │      ├── cleaner_service_areas
  │      ├── cleaner_availability
  │      ├── cleaner_time_off
  │      ├── cleaner_verifications
  │      ├── payouts
  │      └── reviews
  │
  ├── addresses
  ├── notifications
  ├── messages
  └── support_tickets


customer
  │
  ├── addresses
  ├── bookings
  ├── recurring_bookings
  ├── favorite_cleaners
  ├── payments
  └── reviews


cleaner
  │
  ├── services
  ├── service_areas
  ├── availability
  ├── time_off
  ├── bookings
  ├── payouts
  ├── reviews
  └── verifications


booking
  │
  ├── booking_items
  ├── booking_status_history
  ├── payment
  ├── refund
  ├── payout
  ├── review
  ├── dispute
  ├── conversation
  └── booking_financials
```

---

# 42. Booking State Machine

Bookings should follow controlled state transitions.

```text
REQUESTED
    │
    ├──→ REJECTED
    │
    └──→ CONFIRMED
            │
            ├──→ CANCELLED
            │
            └──→ CLEANER_EN_ROUTE
                      │
                      └──→ IN_PROGRESS
                                │
                                └──→ COMPLETED
```

A booking should not be able to move arbitrarily between statuses.

For example:

```text
COMPLETED → REQUESTED
```

should never be allowed.

---

# 43. Important Database Constraints

## Users

```text
users.email UNIQUE
users.phone UNIQUE
```

## Cleaner Services

```text
UNIQUE(cleaner_id, service_id)
```

## Favorites

```text
UNIQUE(customer_id, cleaner_id)
```

## Reviews

```text
UNIQUE(booking_id, customer_id)
```

## Promotion Usage

Prevent a customer from exceeding the promotion's usage limit.

## Booking Conflicts

The database/application must prevent overlapping confirmed bookings for the same cleaner.

PostgreSQL exclusion constraints are recommended for robust time-range conflict prevention.

---

# 44. Indexing Strategy

Important indexes include:

```text
users(email)

users(phone)

cleaner_profiles(user_id)

cleaner_profiles(verification_status)

cleaner_profiles(is_accepting_bookings)

cleaner_services(cleaner_id)

cleaner_services(service_id)

cleaner_service_areas(cleaner_id)

bookings(customer_id)

bookings(cleaner_id)

bookings(status)

bookings(scheduled_start_at)

bookings(cleaner_id, scheduled_start_at)

payments(booking_id)

payments(provider_payment_id)

payouts(cleaner_id)

reviews(cleaner_id)

notifications(user_id, read_at)

messages(conversation_id, created_at)
```

For location searches, create a PostGIS spatial index.

---

# 45. Geographic Search

The platform's cleaner discovery system should support queries such as:

```text
Find all verified cleaners
within 10 km
of the customer's location
who provide Deep Cleaning
and are available on Saturday at 10:00 AM.
```

Conceptually:

```text
Customer Location
       │
       ▼
Geospatial Search
       │
       ▼
Nearby Cleaners
       │
       ▼
Service Filter
       │
       ▼
Availability Filter
       │
       ▼
Rating / Price Ranking
       │
       ▼
Search Results
```

---

# 46. Search Ranking

The initial ranking algorithm can use:

```text
Search Score =
    Distance Score
  + Rating Score
  + Availability Score
  + Completed Jobs Score
  + Verification Score
```

Example weighting:

```text
Distance       30%
Rating         25%
Availability   20%
Experience     15%
Verification   10%
```

The weights should eventually be configurable.

---

# 47. Data Security

Sensitive information should never be unnecessarily stored in the application database.

### Do Not Store Directly

* Raw credit card numbers
* CVV codes
* Passwords
* Full bank account credentials
* Government ID images without appropriate security controls

### Store

* Password hashes
* Payment provider customer IDs
* Payment provider account IDs
* Secure object-storage references
* Verification status

---

# 48. File Storage

Files should be stored in secure object storage such as:

```text
AWS S3
Google Cloud Storage
Azure Blob Storage
```

PostgreSQL stores metadata:

```text
file_id
storage_key
file_type
owner_id
created_at
```

Files may include:

* Profile photos
* Verification documents
* Review images
* Dispute evidence
* Cleaning before/after photos

---

# 49. Recommended Database Architecture

For the MVP:

```text
                    ┌─────────────────┐
                    │   Mobile App    │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │    API Server   │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
       ┌──────▼──────┐ ┌────▼─────┐ ┌─────▼─────┐
       │ PostgreSQL  │ │  Redis   │ │ Object    │
       │             │ │          │ │ Storage   │
       └─────────────┘ └──────────┘ └───────────┘
                             │
                    ┌────────▼────────┐
                    │ External APIs   │
                    │                 │
                    │ Payments        │
                    │ Maps            │
                    │ SMS             │
                    │ Email           │
                    └─────────────────┘
```

---

# 50. MVP Tables

The first release does not need every table described above.

The recommended MVP database consists of:

```text
users
customer_profiles
cleaner_profiles
addresses
cleaner_service_areas
service_categories
services
cleaner_services
cleaner_availability
cleaner_time_off
bookings
booking_items
booking_status_history
payments
payouts
reviews
review_responses
conversations
conversation_participants
messages
notifications
user_devices
cleaner_verifications
disputes
audit_logs
```

---

# 51. Phase 2 Tables

After validating the MVP:

```text
cleaning_companies
company_members
recurring_bookings
service_addons
refunds
cleaner_payment_accounts
promotions
promotion_usage
fee_rules
booking_financials
verification_documents
dispute_evidence
support_tickets
```

---

# 52. Recommended Data Ownership

```text
CUSTOMER
  ├── Profile
  ├── Addresses
  ├── Bookings
  ├── Payments
  ├── Favorites
  └── Reviews

CLEANER
  ├── Profile
  ├── Services
  ├── Service Areas
  ├── Availability
  ├── Verification
  ├── Bookings
  ├── Earnings
  └── Reviews

PLATFORM
  ├── Users
  ├── Services
  ├── Bookings
  ├── Payments
  ├── Payouts
  ├── Disputes
  ├── Promotions
  ├── Fees
  └── Audit Logs
```

---

# 53. Important Design Principle

The most important principle for this application is:

> **Never rely on current profile information to reconstruct historical transactions.**

For example, if a cleaner changes their price from `$80` to `$100`, an old booking must still show the original `$80` price.

Therefore, booking records should store snapshots of:

* Service name
* Service price
* Add-on price
* Platform fee
* Discount
* Tax
* Customer total
* Cleaner earnings
* Currency

This makes the financial history immutable and auditable.

---

# 54. Recommended MVP Database

For the initial launch, the core data model can be summarized as:

```text
                    USERS
                      │
          ┌───────────┴───────────┐
          │                       │
       CUSTOMER                 CLEANER
          │                       │
       ADDRESSES          ┌────────┼─────────┐
          │               │        │         │
          │            SERVICES  AREAS   AVAILABILITY
          │               │
          └───────┬───────┘
                  │
               BOOKINGS
                  │
       ┌──────────┼───────────┐
       │          │           │
    PAYMENTS    REVIEWS    MESSAGES
       │
    PAYOUTS
```

This model provides a strong foundation for launching CleanConnect in one local market and subsequently expanding to additional locations, cleaners, customers, and service categories.
