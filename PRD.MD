# Product Requirements Document (PRD)

## Product Name

**CleanConnect** — Local Home Cleaning Marketplace

## 1. Product Overview

CleanConnect is a mobile and web application that connects homeowners with local professional cleaners.

The platform allows customers to:

- Find cleaners operating in their area
- View cleaner profiles, ratings, reviews, and pricing
- Request one-time or recurring cleaning services
- Schedule a preferred date and time
- Communicate with cleaners
- Pay securely through the application
- Rate and review cleaners after a completed service

Cleaners can use the platform to:

- Create professional profiles
- Define their service areas
- Set availability and pricing
- Receive cleaning requests
- Manage bookings
- Communicate with customers
- Get paid

---

## 2. Problem Statement

Finding a reliable cleaner can be difficult.

Customers often rely on:

- Word of mouth
- Social media
- WhatsApp groups
- Classified advertisements
- Informal recommendations

These methods make it difficult to verify a cleaner's:

- Reliability
- Availability
- Pricing
- Experience
- Previous customer satisfaction

CleanConnect solves this by providing a trusted marketplace where customers and cleaners can connect, communicate, schedule services, and complete payments.

---

## 3. Goals

### Primary Goals

1. Help customers quickly find cleaners near their location.
2. Make it easy to compare cleaners.
3. Allow customers to book cleaning services online.
4. Provide transparent pricing.
5. Help cleaners acquire new customers.
6. Build trust through profiles, reviews, ratings, and verification.
7. Provide a simple booking and payment experience.

### Secondary Goals

- Support recurring cleaning appointments.
- Allow cleaners to manage their schedules.
- Reduce cancellations and missed appointments.
- Create a reliable source of income for local cleaners.

---

## 4. Target Users

### Customer

A homeowner, renter, landlord, property manager, or other person looking for cleaning services.

### Cleaner

An independent cleaner or professional cleaning company offering residential cleaning services.

### Administrator

The business operator responsible for managing the marketplace, users, payments, disputes, verification, and platform settings.

---

# 5. Core Features

## 5.1 Customer Registration

Customers can create an account using:

- Email
- Phone number
- Social login
- Password

The customer should provide:

- Full name
- Phone number
- Email address
- Home/service address
- Profile photo (optional)

---

## 5.2 Location-Based Cleaner Search

The application should identify cleaners available within the customer's selected area.

Customers can search using:

- Current location
- Address
- Neighborhood
- Postal/ZIP code
- Map

Search results should show cleaners based on proximity and availability.

### Search Filters

Customers should be able to filter by:

- Distance
- Price
- Rating
- Availability
- Cleaning type
- Number of bedrooms
- Number of bathrooms
- One-time cleaning
- Recurring cleaning
- Verified cleaners

---

## 5.3 Cleaner Profiles

Each cleaner should have a public profile containing:

- Name
- Profile photo
- Business name
- Description
- Service area
- Years of experience
- Services offered
- Starting price
- Availability
- Customer rating
- Number of completed jobs
- Customer reviews
- Verification status
- Cancellation policy

### Example

**Maria's Cleaning Services**

- ⭐ 4.9/5
- 127 completed cleanings
- Verified Cleaner
- Starting at $80
- Available in your area

---

# 6. Cleaning Services

Cleaners can specify the services they provide.

## Standard Cleaning

- Dusting
- Vacuuming
- Sweeping
- Mopping
- Bathroom cleaning
- Kitchen cleaning
- Surface cleaning
- Trash removal

## Deep Cleaning

- Detailed kitchen cleaning
- Detailed bathroom cleaning
- Baseboards
- Doors
- Windows
- Hard-to-reach areas

## Optional Services

- Laundry
- Ironing
- Fridge cleaning
- Oven cleaning
- Interior window cleaning
- Move-in cleaning
- Move-out cleaning
- Airbnb/short-term rental cleaning

---

# 7. Booking System

Customers can select:

1. Cleaner
2. Service type
3. Property size
4. Number of bedrooms
5. Number of bathrooms
6. Additional services
7. Date
8. Time
9. Address
10. Special instructions

The application calculates the estimated price before the customer confirms.

### Booking Statuses

- Requested
- Pending cleaner confirmation
- Confirmed
- Cleaner en route
- Cleaning in progress
- Completed
- Cancelled
- Disputed

---

# 8. Recurring Cleaning

Customers should be able to schedule recurring services.

Options:

- Weekly
- Every two weeks
- Every four weeks
- Custom schedule

Customers should be able to modify or cancel future appointments.

---

# 9. Cleaner Availability

Cleaners should have a calendar where they can define:

- Working days
- Working hours
- Days off
- Vacation periods
- Existing bookings

The system should prevent customers from booking a cleaner during unavailable times.

---

# 10. Messaging

Customers and cleaners should be able to communicate through an in-app messaging system.

Features should include:

- Text messages
- Booking-related messages
- Photo attachments
- Automated booking notifications
- Push notifications

For safety and privacy, personal contact information should not be unnecessarily exposed.

---

# 11. Payments

Customers should be able to pay securely through the application.

Supported payment methods could include:

- Credit/debit cards
- Apple Pay
- Google Pay
- Other locally supported payment methods

The platform should support:

- Payment authorization
- Payment capture
- Refunds
- Cleaner payouts
- Platform commission
- Receipts

### Example

```text
Cleaning service:       $100
Platform fee:             $10
-----------------------------
Customer pays:          $110

Cleaner receives:        $90
Platform revenue:        $10
