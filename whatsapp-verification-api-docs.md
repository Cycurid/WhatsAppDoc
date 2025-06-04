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

## Phone Number Formatting

The API now supports enhanced phone number formatting with automatic E.164 conversion:

### Supported Input Formats
- **With Country Code**: `+1234567890`, `+442012345678`, `+2348031234567`
- **Without Country Code**: `1234567890`, `2012345678`, `8031234567`
- **With Leading Zeros**: `08031234567` (automatically cleaned)
- **With Spaces/Dashes**: `+1 (234) 567-890` (automatically cleaned)

### Country Code Support
- Use the optional `countryCode` parameter for better international formatting
- Supports all ISO 3166-1 alpha-2 country codes (US, CA, GB, NG, DE, FR, etc.)
- Defaults to "CA" (Canada) if not provided
- Examples:
  - `"8031234567"` with `"countryCode": "NG"` → `+2348031234567`
  - `"2012345678"` with `"countryCode": "GB"` → `+442012345678`
  - `"6043554911"` with `"countryCode": "US"` → `+16043554911`

### Validation
- Phone numbers are validated using international standards
- Invalid formats will be rejected with a 400 error
- Numbers must be valid for the specified country

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
  "phoneNumber": "+1234567890",
  "countryCode": "US"
}
```

**Request Body Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| phoneNumber | string | Yes | Phone number in various formats (e.g., +1234567890, 1234567890). The API will automatically format it to E.164 |
| countryCode | string | No | 2-character ISO country code (e.g., "US", "CA", "GB", "NG"). Used for better phone number formatting when no country code is provided in phoneNumber. Defaults to "CA" |

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
  - **Body (Invalid Phone Number Format):**
  ```json
  {
    "success": false,
    "message": "Phone number is too long for +1. Expected 10 digits after country code, but got 11 digits (1 extra). Number: 60435549280",
    "errors": ["Phone number is too long for +1. Expected 10 digits after country code, but got 11 digits (1 extra). Number: 60435549280"]
  }
  ```
  
  - **Body (Invalid WhatsApp Recipient - Error 63024):**
  ```json
  {
    "success": false,
    "message": "Invalid WhatsApp recipient. The phone number may not have WhatsApp installed or may not be reachable.",
    "error": "Detailed Twilio error message",
    "errorCode": "63024",
    "messageSid": "MMxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
  ```
  
  - **Body (WhatsApp Delivery Failed - Error 63016):**
  ```json
  {
    "success": false,
    "message": "WhatsApp message could not be delivered. The recipient may have blocked your number or does not have WhatsApp.",
    "error": "Detailed Twilio error message",
    "errorCode": "63016",
    "messageSid": "MMxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
  ```
  
  - **Body (WhatsApp Not Available - Error 21608):**
  ```json
  {
    "success": false,
    "message": "WhatsApp is not available for this phone number.",
    "error": "Detailed Twilio error message",
    "errorCode": "21608",
    "messageSid": "MMxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
  ```
  
  - **Body (Invalid Phone Number - Error 21211):**
  ```json
  {
    "success": false,
    "message": "Invalid phone number format for WhatsApp.",
    "error": "Detailed Twilio error message",
    "errorCode": "21211",
    "messageSid": "MMxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
  ```
  
  - **Body (Validation Error):**
  ```json
  {
    "success": false,
    "message": "Validation failed",
    "errors": ["phoneNumber - Phone number is required"]
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
    "message": "Failed to send WhatsApp message",
    "error": "Internal server error details"
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
  "phoneNumber": "+1234567890",
  "countryCode": "US"
}
```

**Request Body Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| phoneNumber | string | Yes | Phone number in various formats (e.g., +1234567890, 1234567890). The API will automatically format it to E.164 |
| countryCode | string | No | 2-character ISO country code (e.g., "US", "CA", "GB", "NG"). Used for better phone number formatting when no country code is provided in phoneNumber. Defaults to "CA" |

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
  "countryCode": "US",
  "code": 123456
}
```

**Request Body Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| phoneNumber | string | Yes | Phone number in various formats (e.g., +1234567890, 1234567890). The API will automatically format it to E.164 |
| countryCode | string | No | 2-character ISO country code (e.g., "US", "CA", "GB", "NG"). Used for better phone number formatting when no country code is provided in phoneNumber. Defaults to "CA" |
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
   - Phone number (automatically formatted to E.164)
   - Verification code
   - User ID
   - Message SID (from WhatsApp/Twilio)
   - Created timestamp
   - Updated timestamp

4. **Duplicate Handling:** If a verification code is requested for a phone number that already has a pending verification, the system updates the existing record with a new code and timestamp.

5. **Security:** Upon successful verification, the verification record is immediately deleted from the database to prevent replay attacks.

6. **Delivery Validation:** The API validates actual message delivery, not just successful API calls:
   - Checks immediate delivery status
   - Waits 2 seconds and re-checks for delayed errors (like WhatsApp availability)
   - Only returns success if the message can actually be delivered
   - Returns specific error codes and messages for different failure types

## Error Handling

### Phone Number Validation Errors
The API provides specific error messages for phone number format issues:
- **Too many digits:** "Phone number is too long for +1. Expected 10 digits after country code, but got 11 digits (1 extra). Number: 60435549280"
- **Too few digits:** "Phone number is too short for +1. Expected 10 digits after country code, but got 9 digits (missing 1). Number: 604355492"
- **Invalid format:** Country-specific validation messages based on libphonenumber-js

### WhatsApp Delivery Errors
The API catches and reports specific WhatsApp delivery failures:

| Error Code | Description | User Action |
|------------|-------------|-------------|
| **63024** | Invalid WhatsApp recipient | The phone number may not have WhatsApp installed or may not be reachable |
| **63016** | Message undelivered | The recipient may have blocked your number or does not have WhatsApp |
| **21211** | Invalid phone number format | Use a valid phone number format |
| **21608** | WhatsApp not available | WhatsApp is not available for this phone number |

### Error Response Format
All error responses include:
- `success: false`
- `message`: User-friendly error description
- `error`: Technical error details (when available)
- `errorCode`: Twilio error code (when available)
- `messageSid`: Twilio message ID (when available)

## Example Usage

### Sending a Verification Code

#### Using cURL:
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/sendVerificationCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+1234567890",
    "countryCode": "US"
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
        phoneNumber: '+1234567890',
        countryCode: 'US'
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
    "phoneNumber": "+1234567890",
    "countryCode": "US"
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
        phoneNumber: '+1234567890',
        countryCode: 'US'
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
    "countryCode": "US",
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
        countryCode: 'US',
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

## International Examples

### Nigerian Number Example
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/sendVerificationCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "8031234567",
    "countryCode": "NG"
  }'
```
This will format to `+2348031234567` and send to Nigeria.

### UK Number Example
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/sendVerificationCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "2012345678",
    "countryCode": "GB"
  }'
```
This will format to `+442012345678` and send to the UK.

### Backward Compatibility
The API maintains full backward compatibility. Existing requests without the `countryCode` parameter will continue to work:
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/sendVerificationCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{
    "phoneNumber": "+1234567890"
  }'
```

## Error Examples

### Invalid Phone Number Format
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/sendVerificationCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber": "+160435549110"}'

# Response:
{
  "success": false,
  "message": "Phone number is too long for +1. Expected 10 digits after country code, but got 11 digits (1 extra). Number: 60435549110",
  "errors": ["Phone number is too long for +1. Expected 10 digits after country code, but got 11 digits (1 extra). Number: 60435549110"]
}
```

### WhatsApp Not Available (Error 63024)
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/sendVerificationCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber": "+16043554911"}'

# Response:
{
  "success": false,
  "message": "Invalid WhatsApp recipient. The phone number may not have WhatsApp installed or may not be reachable.",
  "error": "The 'To' phone number: +16043554911, is not currently reachable via WhatsApp",
  "errorCode": "63024",
  "messageSid": "MMe95b5346055434981d1cbdb87589617a"
}
```

### Wrong Country Code
```bash
curl -X POST https://api2.cycurid.com/v2/public/whatsapp/sendVerificationCode \
  -u "API_KEY:API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"phoneNumber": "6043554911", "countryCode": "GB"}'

# Response:
{
  "success": false,
  "message": "Invalid phone number \"6043554911\" has invalid format for country code +44",
  "errors": ["Invalid phone number \"6043554911\" has invalid format for country code +44"]
}
```

## Rate Limiting

Please note that rate limiting may be applied to these endpoints to prevent abuse. Check the API response headers for rate limit information.