# 📘 **CASP AI — User Sign-Up & Authentication Flow (For UI Developers)**

*Version 1.0 — Frontend Integration Guide*

---

# 🔐 **1. Overview**

The CASP AI authentication system follows this flow:

1. **User enters email + password → Sign Up**
2. **Backend sends a verification email**
3. **User clicks verification link**
4. **Email is verified → user can now log in**
5. **User logs in through `/token` endpoint (OAuth2)**

This document describes:

* Endpoints
* Request / response structures
* UI actions
* Error handling
* Expected flow

---

# 🧭 **2. Complete Workflow Diagram (Frontend Perspective)**

### **Step 1 — Sign Up (POST /signup)**

User submits: `email`, `password`, `first_name`

⬇️
Backend creates account + sends verification email

⬇️
UI shows:
**“Check your email for a verification link.”**

---

### **Step 2 — Email Verification (GET /verify-email?token=XYZ)**

When user clicks link → UI should redirect to a "Verification Successful" page.

⬇️
After verification UI should show:
**“Your email is verified. Please log in.”**

---

### **Step 3 — Login (POST /{resource}/users/token)**

User enters:
`username` + `password`

Backend returns:

* `access_token`
* `token_type`
* `resource`
* `username`

UI stores token in localStorage / sessionStorage.

---

# 🚀 **3. API Endpoints for UI Integration**

---

## ✅ **3.1 Sign Up Endpoint**

### **POST /signup**

### **Request Body**

```json
{
  "email": "user@example.com",
  "password": "SomePassword123",
  "first_name": "John"
}
```

### **Successful Response (200)**

```json
{
  "message": "Verification email sent. Please check your inbox."
}
```

### **Errors**

| Status | Meaning                  |
| ------ | ------------------------ |
| 400    | Email already registered |
| 500    | Unexpected server error  |

---

## 🔗 **3.2 Email Verification Endpoint**

### **GET /verify-email?token=UUID-HERE**

UI does NOT need to send JSON.
Just navigate user to this link.

### **Successful Response**

```json
{
  "message": "Email verified successfully. You may now log in."
}
```

UI should then navigate to the **Login Page**.

### Possible Errors

| Status | Meaning                  |
| ------ | ------------------------ |
| 400    | Token invalid or expired |
| 500    | Server error             |

---

## 🔐 **3.3 Login Endpoint (Existing)**

### **POST /{resource}/users/token**

This uses OAuth2PasswordRequestForm, so UI must send:

### **Form Data (NOT JSON)**

```
username=user@example.com
password=thepassword
```

### **Example Request (JS Fetch)**

```javascript
const formData = new FormData();
formData.append("username", email);
formData.append("password", password);

fetch(`/auth/users/token`, {
  method: "POST",
  body: formData
})
```

### **Successful Response**

```json
{
  "access_token": "<JWT_TOKEN>",
  "token_type": "bearer",
  "resource": "auth",
  "username": "user@example.com",
  "expires_in": "2025-02-20T08:00:00Z"
}
```

### UI must store:

* `access_token` (for Authorization: Bearer)
* `username`
* `resource`

### Errors

| Status | Meaning                     |
| ------ | --------------------------- |
| 401    | Incorrect username/password |
| 401    | Email not verified          |
| 500    | Server error                |

---

# 🧑‍💻 **4. UI Implementation Responsibilities**

### ✔️ **4.1 Sign-Up Page**

Fields:

* First Name
* Email
* Password
* Confirm Password

Actions:

* Call `/signup`
* On success → show “Check your inbox to verify your email”

---

### ✔️ **4.2 Email Verification Page**

UI receives redirect from backend link.

Example URL:

```
https://app.caspai.in/verify?token=12345
```

UI must:

1. Extract `token`
2. Call:

```
GET /verify-email?token={token}
```

3. Show:

   * ✔️ Success → “Email verified. Please log in.”
   * ❌ Error → “Invalid or expired verification link.”

---

### ✔️ **4.3 Login Page**

Standard login form.

On success:

* Save `JWT token` from `access_token`
* Redirect user to dashboard
* Add Authorization header to all future API calls:

```
Authorization: Bearer <token>
```

---

# 🔧 **5. Token Storage Guidelines (UI)**

Frontend should store token in:

**Preferred:**

* `sessionStorage` → safer
* or `in-memory storage` if SPA

**Avoid unless necessary:**

* `localStorage` (XSS risk)

---

# 🧪 **6. Testing Checklist for UI Developer**

| Feature                   | Expected Behavior                      |
| ------------------------- | -------------------------------------- |
| Sign-up                   | Should receive “Email sent” message    |
| Email verification        | Redirect shows “Verified successfully” |
| Login before verification | Should show error “Email not verified” |
| Login after verification  | Should get access token                |
| API calls                 | Must include JWT header                |

---

# 📁 **7. Reference Verification Email Format**

This is the template user receives in inbox:

* CASP Logo
* Welcome Message
* “Verify Email” Button
* Fallback link
* CASP highlights & links

---

# 📘 **8. Summary for UI Team**

The UI needs to implement:

1. **POST /signup**
2. Confirmation screen
3. Email verification landing page + API call
4. Login page using OAuth2 form-data
5. Store JWT token
6. Attach token to all protected API calls
