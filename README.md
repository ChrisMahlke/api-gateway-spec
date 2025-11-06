# API Gateway Specification

<p align="center">
  <img src="https://img.shields.io/badge/Google_API_Gateway-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white" alt="Google API Gateway">
  <img src="https://img.shields.io/badge/OpenAPI-6BA539?style=for-the-badge&logo=openapiinitiative&logoColor=white" alt="OpenAPI">
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge" alt="License: MIT">
</p>

OpenAPI 2.0 (Swagger) specification for the **Digital Doppelganger** API Gateway. This specification defines the API Gateway configuration, routes, authentication, and backend service integration.

> **Part of the Demographic Doppelgänger Project**  
> 🌐 [Live Application](https://demographic-doppelganger-71027948544.us-west1.run.app/) | [Frontend Repository](https://github.com/ChrisMahlke/doppelganger) | [Backend Engine](https://github.com/ChrisMahlke/doppelganger-engine)

## Overview

This repository contains the OpenAPI specification that configures Google Cloud API Gateway for the Demographic Doppelganger application. The API Gateway serves as the public-facing entry point, handling authentication, request routing, and CORS for the frontend application.

## Architecture

The API Gateway architecture:

```
Frontend (React)
    ↓
Google API Gateway (This Spec)
    ↓ (IAM Authentication)
Python Engine Service (Cloud Run)
    ↓
Firestore + Census API + Gemini AI
```

## Specification Details

### Backend Configuration

- **Backend Service**: `doppelganger-engine` (Cloud Run)
- **Authentication**: IAM-based (API Gateway service account)
- **Timeout**: 120 seconds (matches backend service timeout)
- **CORS**: Handles OPTIONS preflight requests automatically

### Endpoints

#### POST `/find-twin`

Finds demographic doppelgangers for a given ZIP code.

**Authentication**: Requires API key in `x-api-key` header

**Request:**
```json
{
  "zip_code": "90210"
}
```

**Response:**
```json
{
  "demographics": { ... },
  "profile": { ... },
  "doppelgangers": [ ... ]
}
```

#### OPTIONS `/find-twin`

CORS preflight handler. No authentication required.

**Response:** 204 No Content (with CORS headers)

### Security

- **API Key Authentication**: Required for POST requests via `x-api-key` header
- **CORS Preflight**: OPTIONS requests don't require authentication
- **Backend Authentication**: API Gateway authenticates to Cloud Run using IAM

## Deployment

### Prerequisites

1. Google Cloud Project with API Gateway API enabled
2. Backend Cloud Run service (`doppelganger-engine`) deployed
3. API Gateway service account with `roles/run.invoker` permission on backend service
4. API keys created and configured in Google Cloud Console

### Deploy API Gateway

1. **Create API Gateway API** (if not exists):

   ```bash
   gcloud api-gateway apis create doppelganger-gateway \
     --project=PROJECT_ID \
     --display-name="Digital Doppelganger Gateway"
   ```

2. **Create API Config**:

   ```bash
   gcloud api-gateway api-configs create v1 \
     --api=doppelganger-gateway \
     --openapi-spec=openapi.yaml \
     --project=PROJECT_ID
   ```

3. **Create or Update Gateway**:

   ```bash
   # Create gateway
   gcloud api-gateway gateways create doppelganger-gateway \
     --api=doppelganger-gateway \
     --api-config=v1 \
     --location=us-central1 \
     --project=PROJECT_ID
   
   # Or update existing gateway
   gcloud api-gateway gateways update doppelganger-gateway \
     --api=doppelganger-gateway \
     --api-config=v1 \
     --location=us-central1 \
     --project=PROJECT_ID
   ```

4. **Configure IAM Permissions**:

   ```bash
   # Get API Gateway service account
   GATEWAY_SA=$(gcloud api-gateway gateways describe doppelganger-gateway \
     --location=us-central1 \
     --project=PROJECT_ID \
     --format="value(serviceAccount)")
   
   # Grant permission to invoke backend service
   gcloud run services add-iam-policy-binding doppelganger-engine \
     --region=us-central1 \
     --member="serviceAccount:${GATEWAY_SA}" \
     --role="roles/run.invoker" \
     --project=PROJECT_ID
   ```

5. **Create and Configure API Keys**:

   - Go to [Google Cloud Console → APIs & Services → Credentials](https://console.cloud.google.com/apis/credentials)
   - Create API Key
   - Restrict key to API Gateway service
   - Configure referrer restrictions (frontend URL)
   - Use the key in frontend application

## Configuration Details

### Backend Service URL

The `x-google-backend` configuration points to the Cloud Run service:

```yaml
x-google-backend:
  address: https://doppelganger-engine-5znouwfmaa-uc.a.run.app/find-twin
  deadline: 120
```

**Important**: Update the `address` field if your Cloud Run service URL changes.

### Timeout Configuration

The `deadline` field (120 seconds) must match:
- Backend Cloud Run service timeout (`--timeout=120s`)
- Frontend request timeout expectations

### CORS Handling

- **OPTIONS requests**: No authentication required (handled by backend CORS)
- **POST requests**: Require API key authentication
- CORS headers are automatically added by the backend service

## Updating the Specification

1. Edit `openapi.yaml`
2. Create new API config version:

   ```bash
   gcloud api-gateway api-configs create v2 \
     --api=doppelganger-gateway \
     --openapi-spec=openapi.yaml \
     --project=PROJECT_ID
   ```

3. Update gateway to use new config:

   ```bash
   gcloud api-gateway gateways update doppelganger-gateway \
     --api=doppelganger-gateway \
     --api-config=v2 \
     --location=us-central1 \
     --project=PROJECT_ID
   ```

## Security Considerations

### Making This Repository Public

**Yes, this repository is safe to make public** because:

- ✅ **No Secrets**: The OpenAPI spec contains no API keys, passwords, or credentials
- ✅ **Public URLs**: Cloud Run service URLs are public by design (they're endpoints)
- ✅ **Standard Spec**: The security definitions are standard OpenAPI patterns
- ✅ **Configuration Only**: This is configuration, not executable code with secrets

**What's Included:**
- Backend service URL (public endpoint)
- API endpoint structure (public API)
- Security definition structure (standard OpenAPI)
- Request/response schemas (public API contract)

**What's NOT Included:**
- Actual API keys (managed separately in Google Cloud Console)
- Service account credentials (managed via IAM)
- Database connection strings (not in this spec)
- Any secrets or credentials

### Best Practices

1. **API Keys**: Manage API keys through Google Cloud Console, not in code
2. **Service URLs**: If you change service URLs, update the spec before deploying
3. **Version Control**: Use versioned API configs for safe rollbacks
4. **IAM Permissions**: Regularly audit IAM permissions on backend services

## Project Structure

```
api-gateway-spec/
├── openapi.yaml    # OpenAPI 2.0 specification
└── README.md       # This file
```

## Related Repositories

This specification is part of the **Demographic Doppelgänger** project:

- 🌐 **Live Application**: [https://demographic-doppelganger-71027948544.us-west1.run.app/](https://demographic-doppelganger-71027948544.us-west1.run.app/)
- 🎨 **Frontend Repository**: [doppelganger](https://github.com/ChrisMahlke/doppelganger) - React/TypeScript frontend
- 🐍 **Backend Engine**: [doppelganger-engine](https://github.com/ChrisMahlke/doppelganger-engine) - Python Flask service
- 🔧 **Node.js API** (Deprecated): [doppelganger-api](https://github.com/ChrisMahlke/doppelganger-api) - Legacy Node.js gateway service

## Troubleshooting

### Common Issues

1. **503 Service Unavailable**:
   - Check API Gateway service account has `roles/run.invoker` on backend service
   - Verify backend service is running and healthy
   - Check backend service URL in `openapi.yaml` is correct

2. **403 Forbidden (API Key)**:
   - Verify API key is valid and not expired
   - Check API key restrictions (referrer, API targets)
   - Ensure API key is included in `x-api-key` header

3. **504 Gateway Timeout**:
   - Verify `deadline` in spec matches backend timeout
   - Check backend service is processing requests successfully
   - Review backend logs for slow operations

4. **CORS Errors**:
   - Verify OPTIONS endpoint is configured (no auth required)
   - Check backend service handles OPTIONS requests
   - Ensure frontend URL is in API key referrer restrictions

## References

- [Google Cloud API Gateway Documentation](https://cloud.google.com/api-gateway/docs)
- [OpenAPI Specification](https://swagger.io/specification/)
- [API Gateway x-google-backend Extension](https://cloud.google.com/api-gateway/docs/openapi-specification#backend-configuration)

## License

MIT License - see LICENSE file for details

