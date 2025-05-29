# WhatsApp Verification API Documentation

## Overview

The WhatsApp Verification API provides endpoints for sending and verifying phone numbers using WhatsApp messaging. This service allows users to verify their phone numbers through a 6-digit verification code sent via WhatsApp.

## Authentication

All endpoints require Basic Authentication using the merchant's API key and secret.

```bash
curl -u "API_KEY:API_SECRET"
```


**Alternative: Using Authorization header :**
```bash
# First encode your credentials to base64
# Format: merchantApiKey:merchantSecret
echo -n "your-api-key:your-secret" | base64

# Then use:
Authorization: Basic <base64-encoded-credentials>
```

**Important:** 
- Ensure your API key and secret are correct and active
- Make sure there are no trailing spaces

## Endpoints

### 1. Send Verification Code

Sends a 6-digit verification code to the specified phone number via WhatsApp.

**Endpoint:** `POST /v2/public/whatsapp/sendVerificationCode`

**Headers:**
```
Content-Type: application/json
Authorization: Basic API_KEY:API_SECRET
```

**Request Body:**
```json
{
  "phoneNumber": "+1234567890"
}
```

**Request Body Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| phoneNumber | string | Yes | Phone number in E.164 format (e.g., +1234567890). Must match regex: `^\+[1-9]\d{1,14}$` |

**Success Response:**
- **Status Code:** 200 OK
- **Body:**
```json
{
  "success": true,
  "message": "Verification code sent successfully",
  "messageSid": "SMXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}
```

**Error Responses:**

- **Status Code:** 400 Bad Request
  - **Body:**
  ```json
  {
    "success": false,
    "message": "Validation failed",
    "errors": ["phoneNumber - Invalid phone number format"]
  }
  ```

- **Status Code:** 401 Unauthorized
  - **Body:**
  ```json
  {
    "error": "Unauthorized: User not found"
  }
  ```

- **Status Code:** 422 Unprocessable Entity
  - **Body:**
  ```json
  {
    "success": false,
    "message": "Validation failed",
    "errors": "Error message details"
  }
  ```

- **Status Code:** 500 Internal Server Error
  - **Body:**
  ```json
  {
    "success": false,
    "message": "Failed to send SMS",
    "error": "Error details"
  }
  ```

### 2. Resend Verification Code

Resends a new 6-digit verification code to the specified phone number via WhatsApp. This endpoint uses the same controller as the send verification code endpoint.

**Endpoint:** `POST /v2/public/whatsapp/resendVerificationCode`

**Headers:**
```
Content-Type: application/json
Authorization: Basic API_KEY:API_SECRET
```

**Request Body:**
```json
{
  "phoneNumber": "+1234567890"
}
```

**Request Body Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| phoneNumber | string | Yes | Phone number in E.164 format (e.g., +1234567890). Must match regex: `^\+[1-9]\d{1,14}$` |

**Responses:** Same as Send Verification Code endpoint

### 3. Verify Code

Verifies the 6-digit code sent to the user's phone number. The code must be verified within 5 minutes of being sent.

**Endpoint:** `POST /v2/public/whatsapp/verifyCode`

**Headers:**
```
Content-Type: application/json
Authorization: Basic API_KEY:API_SECRET
```

**Request Body:**
```json
{
  "phoneNumber": "+1234567890",
  "code": 123456
}
```

**Request Body Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| phoneNumber | string | Yes | Phone number in E.164 format (e.g., +1234567890). Must match regex: `^\+[1-9]\d{1,14}$` |
| code | number | Yes | 6-digit verification code. Must be between 100000 and 999999 |

**Success Response:**
- **Status Code:** 200 OK
- **Body:**
```json
{
  "success": true,
  "message": "Verification code is valid and record deleted"
}
```

**Error Responses:**

- **Status Code:** 400 Bad Request
  - **Body (Validation Error):**
  ```json
  {
    "success": false,
    "message": "Validation failed",
    "errors": ["code - Code must be a 6-digit number"]
  }
  ```
  
  - **Body (Invalid/Expired Code):**
  ```json
  {
    "success": false,
    "message": "Invalid phone number, verification code, or code has expired"
  }
  ```

- **Status Code:** 401 Unauthorized
  - **Body:**
  ```json
  {
    "error": "Unauthorized: User not found"
  }
  ```

- **Status Code:** 422 Unprocessable Entity
  - **Body:**
  ```json
  {
    "success": false,
    "message": "Validation failed",
    "errors": "Error message details"
  }
  ```

- **Status Code:** 500 Internal Server Error
  - **Body:**
  ```json
  {
    "success": false,
    "message": "Unable due to an internal error"
  }
  ```

## Implementation Notes

1. **Code Generation:** Verification codes are 6-digit random numbers generated using `Math.floor(100000 + Math.random() * 900000)`.

2. **Code Expiration:** Verification codes expire after 5 minutes. Any attempt to verify an expired code will result in a 400 error.

3. **Database Storage:** The system stores:
   - Phone number
   - Verification code
   - User ID
   - Message SID (from WhatsApp/Twilio)
   - Created timestamp
   - Updated timestamp

4. **Duplicate Handling:** If a verification code is requested for a phone number that already has a pending verification, the system updates the existing record with a new code and timestamp.

5. **Security:** Upon successful verification, the verification record is immediately deleted from the database to prevent replay attacks.

## Example Usage

### Sending a Verification Code

#### Using cURL:
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/sendVerificationCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+1234567890"
  }'
```

#### Using Axios (JavaScript):
```javascript
const axios = require('axios');

const sendVerificationCode = async () => {
  try {
    const response = await axios.post(
      'https://api2.cycurid.com/v2/public/whatsapp/sendVerificationCode',
      {
        phoneNumber: '+1234567890'
      },
      {
        headers: {
          'Content-Type': 'application/json'
        },
        auth: {
          username: 'API_KEY',
          password: 'API_SECRET'
        }
      }
    );
    
    console.log('Success:', response.data);
  } catch (error) {
    console.error('Error:', error.response?.data || error.message);
  }
};

sendVerificationCode();
```

### Resending a Verification Code

#### Using cURL:
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/resendVerificationCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+1234567890"
  }'
```

#### Using Axios (JavaScript):
```javascript
const axios = require('axios');

const resendVerificationCode = async () => {
  try {
    const response = await axios.post(
      'https://api2.cycurid.com/v2/public/whatsapp/resendVerificationCode',
      {
        phoneNumber: '+1234567890'
      },
      {
        headers: {
          'Content-Type': 'application/json'
        },
        auth: {
          username: 'API_KEY',
          password: 'API_SECRET'
        }
      }
    );
    
    console.log('Success:', response.data);
  } catch (error) {
    console.error('Error:', error.response?.data || error.message);
  }
};

resendVerificationCode();
```

### Verifying a Code

#### Using cURL:
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/verifyCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+1234567890",
    "code": 123456
  }'
```

#### Using Axios (JavaScript):
```javascript
const axios = require('axios');

const verifyCode = async () => {
  try {
    const response = await axios.post(
      'https://api2.cycurid.com/v2/public/whatsapp/verifyCode',
      {
        phoneNumber: '+1234567890',
        code: 123456
      },
      {
        headers: {
          'Content-Type': 'application/json'
        },
        auth: {
          username: 'API_KEY',
          password: 'API_SECRET'
        }
      }
    );
    
    console.log('Success:', response.data);
  } catch (error) {
    console.error('Error:', error.response?.data || error.message);
  }
};

verifyCode();
```

## Rate Limiting

Please note that rate limiting may be applied to these endpoints to prevent abuse. Check the API response headers for rate limit information.