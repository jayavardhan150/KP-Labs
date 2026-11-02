# LAB 2 – OTP Verification Bypass

## Overview

The Taskbear registration process allows a user to proceed to account creation when the OTP verification response is modified in the client-side proxy. During testing, an incorrect OTP was rejected by the server with `401 Unauthorized`. After changing the response to the successful verification response and forwarding it to the browser, the registration workflow continued and the lab account was created.

## Steps to Reproduce

1. Open `https://taskbear.com.au/` and navigate to the sign-up page.
2. Enter an email address that you control and click **Send verification code**.
3. Intercept the OTP verification request in Burp Suite:

   ```http
   POST /api/v1/uwa/auth/otp/verify HTTP/2
   Host: master.prod.api.taskbear.com.au
   Content-Type: application/json

   {
     "contact": "YOUR_EMAIL",
     "otp": "VALID_OTP"
   }
   ```

4. Forward the request with the valid OTP and observe the successful response. Note the response status and JSON structure for comparison.
5. Start a fresh registration attempt and request another OTP.
6. Enter an incorrect OTP and intercept the response to the same endpoint.
7. Observe the actual server response:

   ```http
   HTTP/2 401 Unauthorized
   Content-Type: application/json

   {
     "statusCode": 401,
     "message": "OTP invalid or expired"
   }
   ```

8. In Burp Suite, change the intercepted response status from `401 Unauthorized` to `201 Created`.
9. Replace the response body with the successful response structure observed in the valid-OTP test. For example:

   ```http
   HTTP/2 201 Created
   Content-Type: application/json

   {
     "contact": "YOUR_EMAIL",
     "verifiedAt": "TIMESTAMP"
   }
   ```

   Use the email address from the current registration attempt and the response format observed in the successful baseline request.

10. Forward the modified response to the browser.
11. Observe that the browser advances to the next registration step instead of displaying the OTP verification error.
12. Complete the remaining registration fields and submit the account creation form.
13. Confirm that the lab account is created even though the OTP submitted in this test was incorrect.

**Endpoint tested:** `POST https://master.prod.api.taskbear.com.au/api/v1/uwa/auth/otp/verify`

**Observed behavior:** The server's original response rejects the incorrect OTP, but the client-side registration flow accepts the modified success response and continues.

## Proof of Concept (PoC)

The screenshots below show the complete test flow, including registration, OTP submission, the original server response, response modification, and successful progression through account creation.

### PoC Evidence 1

![PoC Evidence 1](evidence/poc-01.png)

### PoC Evidence 2

![PoC Evidence 2](evidence/poc-02.png)

### PoC Evidence 3

![PoC Evidence 3](evidence/poc-03.png)

### PoC Evidence 4

![PoC Evidence 4](evidence/poc-04.png)

### PoC Evidence 5

![PoC Evidence 5](evidence/poc-05.png)

### PoC Evidence 6

![PoC Evidence 6](evidence/poc-06.png)

### PoC Evidence 7

![PoC Evidence 7](evidence/poc-07.png)

### PoC Evidence 8

![PoC Evidence 8](evidence/poc-08.png)

### PoC Evidence 9

![PoC Evidence 9](evidence/poc-09.png)

### PoC Evidence 10

![PoC Evidence 10](evidence/poc-10.png)

## Impact

- OTP verification can be bypassed in the demonstrated registration workflow.
- An account can be created without successfully validating the email address through the OTP process.
- If the same behavior exists in production and the backend does not independently enforce verification, an attacker could register accounts using email addresses they have not verified.
- This may undermine email ownership checks and any security or recovery process that relies on a verified email address.

## Remediation

- Enforce OTP verification on the server. Never rely on a browser-side response, JavaScript state, or client-controlled flag as proof of verification.
- After a valid OTP is checked, generate a cryptographically secure, short-lived verification token on the server.
- Bind the token to the email address and registration session.
- Require the token when creating the account and validate it server-side before accepting registration.
- Make the verification token single-use and invalidate it after successful account creation or expiration.
- Apply rate limits, expiry controls, and attempt limits to OTP generation and verification.
- Add server-side tests confirming that account creation fails when OTP verification has not succeeded.

