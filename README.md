# MyBOQ Community Service plan

## Overview

The Community module is implemented as a **separate Spring Boot service** with its **own dedicated database** (`myboq_community`).
This design ensures clean separation between the core MyBOQ application and the Community feature, while still allowing seamless user authentication.

The Community service **does not manage authentication credentials** on its own.
Instead, it integrates with the existing MyBOQ authentication system by using the `sts_ssrpro.sts_users` table **only for login and registration verification**.

---

## Database Strategy

### Databases Used

1. **sts_ssrpro**

   * Existing MyBOQ application database
   * Contains authentication and core user data
   * Community service accesses **only one table**:

     * `sts_users` (read-only usage)

2. **myboq_community**

   * Dedicated database for Community module
   * Stores all community-related data:

     * Users (`cm_users`)
     * Topics, replies, likes, views
     * Notifications, stats, points, moderation data

This approach avoids mixing community data with the existing MyBOQ schema, which already contains multiple tables and production data.

---

## Authentication & Account Linking Flow

### Key Principle

* **Login credentials (username/password)** are always validated against
  `sts_ssrpro.sts_users`
* **Community profile data** is stored only in
  `myboq_community.cm_users`

There is **no database-level foreign key** between the two databases.
The relationship is validated and enforced at the **service layer**.

---

## Community Registration Flow

### API: `link-myboq-account`

This API is responsible for linking an existing MyBOQ account with the Community system.

### Input

* `username` (MyBOQ login identifier – email or mobile)
* `password`

### Registration Logic

1. Validate credentials against `sts_ssrpro.sts_users`
2. If credentials are valid:

   * Fetch user details from `sts_users`
3. Create a new entry in `myboq_community.cm_users` with default values:

```text
id                → auto-generated
public_handle     → username from sts_users (temporary default)
fk_sts_user_id    → sts_users.user_id
email             → from sts_users
mobile            → from sts_users
full_name         → from sts_users
```

4. If credentials are invalid:

   * Return authentication failure response

This ensures that **only existing MyBOQ users** can register for the Community module.

---

## Public Handle Management

* Initially, `public_handle` is populated using the MyBOQ username
* This value is **temporary and system-generated**
* A separate **Profile Update API** is provided where users can:

  * Update their `public_handle`
  * Customize their community identity (Instagram-style handle)

This ensures:

* No exposure of email or mobile during mentions or tagging
* Clean separation between login identity and public community identity

---

## Community Login Flow

### Login Logic

1. User submits `username` and `password`
2. Credentials are validated against `sts_ssrpro.sts_users`
3. If valid:

   * Check in `myboq_community.cm_users` whether an entry exists where
     `cm_users.fk_sts_user_id = sts_users.user_id`
4. If entry exists:

   * Login successful
5. If entry does not exist:

   * Return message:
     **“Please link your MyBOQ account first.”**

This prevents unauthorized access to Community features without proper account linking.

---

## Data Ownership Rules

* `sts_ssrpro.sts_users`

  * Owned by MyBOQ core application
  * Community service has **read-only access**

* `myboq_community.*`

  * Fully owned and managed by Community service
  * Includes all user-generated content and analytics

---

## Benefits of This Architecture

* Clean separation of concerns
* Zero risk to existing MyBOQ production data
* Scalable and future-proof design
* Easy to migrate Community module independently if required
* Industry-standard approach for modular systems

---

## Database Schema Diagram

![Community Schema](schema_diagram.svg)

---
