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

<img width="1279" height="650" alt="1" src="https://github.com/user-attachments/assets/15531d21-e345-4bbc-8ed2-7c859fa0c68d" />


### PoC Evidence 2

<img width="1277" height="656" alt="2" src="https://github.com/user-attachments/assets/65493a15-a03c-4047-84ae-bc9373a6ee94" />


### PoC Evidence 3

<img width="1269" height="655" alt="3" src="https://github.com/user-attachments/assets/10c3dc12-d54b-4453-a091-9aa3498cae52" />


### PoC Evidence 4

<img width="1248" height="570" alt="4" src="https://github.com/user-attachments/assets/9f691c5e-e2c7-4938-9db6-324c065a3ce0" />


### PoC Evidence 5

<img width="1269" height="653" alt="5" src="https://github.com/user-attachments/assets/89014dd4-2771-4e6a-8100-09e7952df77d" />


### PoC Evidence 6

<img width="1275" height="654" alt="6" src="https://github.com/user-attachments/assets/0786b63a-b58b-4b98-902b-998afb805fb9" />


### PoC Evidence 7

<img width="1275" height="654" alt="6" src="https://github.com/user-attachments/assets/b9b57c21-289f-49fc-891b-39060ea54f32" />


### PoC Evidence 8

<img width="1255" height="570" alt="8" src="https://github.com/user-attachments/assets/f6c49016-4e33-4771-818b-ed67d0f4fc2d" />


### PoC Evidence 9

<img width="1254" height="651" alt="9" src="https://github.com/user-attachments/assets/2c357915-45cc-469e-afa3-fb59462506f0" />


### PoC Evidence 10

<img width="1266" height="648" alt="10" src="https://github.com/user-attachments/assets/7817a168-8e5f-4134-a61f-9993db474d34" />


## Impact

- OTP verification can be bypassed in the demonstrated registration workflow.
- An account can be created without successfully validating the email address through the OTP process.
- If the same behavior exists in production and the backend does not independently enforce verification, an attacker could register accounts using email addresses they have not verified.
- This may undermine email ownership checks and any security or recovery process that relies on a verified email address.

## Remediation

- Enforce OTP verification on the server. Never rely on a browser-side response, JavaScript state, or client-controlled flag as proof of verification.
- Bind the token to the email address and registration session.
- Require the token when creating the account and validate it server-side before accepting registration.
- Make the verification token single-use and invalidate it after successful account creation or expiration.
- Add server-side tests confirming that account creation fails when OTP verification has not succeeded.

