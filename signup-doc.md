Here is a clean **API documentation** for the `/test/signup` endpoint based on the Swagger screenshot you provided — rewritten so you can send it directly to your frontend team.

---

# **Signup or Forgot Password API Documentation**

## **Endpoint**

Swagger link :  https://whitecel.com/docs#/pods/signup_user_test_signup_post

**POST** `https://whitecel.com/test/test/signup`

**POST** `https://whitecel.com/test/set-password`

Registers a new user into the system.

---

## **Request Format**

**Content-Type:** `application/json`

### **Minimum Mandatory Fields**

The frontend **must** send the following:

| Field        | Type   | Required | Description                                                                   |
| ------------ | ------ | -------- | ----------------------------------------------------------------------------- |
| **name**     | string | Yes      | Full name of the user                                                         |
| **mail**     | string | Yes      | Email ID (used as unique identifier)                                          |
| **password** | string | Yes      | Plaintext password (server will hash it internally unless otherwise required) |

---

## **Full Request Body Schema (as per backend)**

The backend accepts a larger object. Non-mandatory fields can be left empty.

```json
{
  "username": "",
  "password": "",
  "hashed_password": "",
  "role": [
    "admin",
    "user",
    "manage"
  ],
  "mail": "",
  "resource": "",
  "name": "",
  "picture": "",
  "subscription": "",
  "expiry": ""
}
```

---

## **Simplified Example Request (Frontend Should Send)**

If your backend handles hashing internally and sets defaults:

```json
{
  "name": "John Doe",
  "mail": "john@example.com",
  "password": "StrongPassword123"
}
```

---

## **Backend Behavior (Assumed / Typical)**

* `username` may auto-generate or mirror the email unless required.
* `hashed_password` should not be supplied by the client.
* `role` defaults to `"user"` unless you allow custom role assignment.
* `resource`, `picture`, `subscription`, and `expiry` are optional metadata fields.

---

## **Validation Requirements (recommended for frontend)**

1. **name**

   * Minimum 2 characters
   * No special characters except spaces

2. **mail**

   * Must be a valid email format

3. **password**

   * Minimum 8 characters
   * Should contain upper/lowercase, number, and symbol (recommended)

---

## **Example Successful Response**

```json
{
  "status": "success",
  "message": "User created successfully",
  "user_id": "<generated_id>"
}
```

