# API Contracts for ViveSphere Bot Manager - Phase 1

**Objective**: Define the API contracts for inter-service communication in Phase 1 of the ViveSphere Bot Manager system, ensuring consistency, reliability, and security across microservices.

## Overview

This document specifies the REST API endpoints for the core microservices identified in Phase 1 of the ViveSphere Bot Manager system. Each endpoint adheres to versioning for backward compatibility, uses JSON for data exchange, and incorporates security measures such as JWT (JSON Web Token) authentication where applicable. The contracts are designed to be implemented using the FastAPI framework in Python, with documentation following OpenAPI/Swagger standards.

## API Versioning

All API endpoints are prefixed with `/api/v1/` to ensure version control and backward compatibility as the system evolves. Future updates or breaking changes will increment the version number (e.g., `/api/v2/`).

## Data Format

- **Request/Response Format**: JSON is used for all request payloads and responses to ensure consistency and ease of parsing across services.
- **Error Handling**: Responses for error conditions will include a standard format with `status`, `error_code`, and `message` fields to facilitate debugging and user feedback.

## Security

- **Authentication**: Endpoints requiring user or service authentication use JWT tokens passed in the `Authorization` header (e.g., `Bearer <token>`). The Authentication Microservice validates tokens before other services process requests.
- **Input Validation**: All inputs are validated to prevent injection attacks and ensure data integrity, following security best practices.

## Microservice API Endpoints

### 1. Authentication Microservice

**Purpose**: Handles user authentication and token management for secure access to the system.

- **Endpoint**: `POST /api/v1/auth/login`
  - **Description**: Authenticates a user and returns a JWT token for subsequent requests.
  - **Parameters**:
    - `username` (string, required): The user's login identifier.
    - `password` (string, required): The user's password.
  - **Response**:
    - Success (200 OK): `{"token": "<jwt_token>"}`
    - Failure (401 Unauthorized): `{"status": "error", "error_code": "AUTH_FAILED", "message": "Invalid credentials"}`
  - **Security**: None (public endpoint for login).

- **Endpoint**: `GET /api/v1/auth/validate`
  - **Description**: Validates a provided JWT token and returns the associated user information if valid.
  - **Parameters**:
    - `token` (string, required): The JWT token to validate, passed in query or header.
  - **Response**:
    - Success (200 OK): `{"status": "valid", "user_id": "<user_id>"}`
    - Failure (401 Unauthorized): `{"status": "error", "error_code": "INVALID_TOKEN", "message": "Token is invalid or expired"}`
  - **Security**: None (public endpoint for validation).

### 2. Bot Manager Microservice

**Purpose**: Manages bot creation, operations, and interactions with the X platform API.

- **Endpoint**: `POST /api/v1/bots/create`
  - **Description**: Creates a new bot with the provided details and credentials.
  - **Parameters**:
    - `bot_name` (string, required): A unique name for the bot.
    - `credentials` (object, required): An object containing bot credentials (e.g., API keys, tokens).
  - **Response**:
    - Success (201 Created): `{"bot_id": "<bot_id>", "status": "created"}`
    - Failure (400 Bad Request): `{"status": "error", "error_code": "INVALID_DATA", "message": "Invalid bot data provided"}`
    - Failure (401 Unauthorized): `{"status": "error", "error_code": "UNAUTHORIZED", "message": "Invalid or missing token"}`
  - **Security**: Requires JWT token in `Authorization` header.

- **Endpoint**: `GET /api/v1/bots/list`
  - **Description**: Retrieves a list of bots associated with the authenticated user.
  - **Parameters**:
    - `user_id` (string, required): The ID of the user whose bots are to be listed (derived from token).
  - **Response**:
    - Success (200 OK): `{"bots": ["<bot_id_1>", "<bot_id_2>", ...]}`
    - Failure (401 Unauthorized): `{"status": "error", "error_code": "UNAUTHORIZED", "message": "Invalid or missing token"}`
  - **Security**: Requires JWT token in `Authorization` header.

### 3. Configuration/Settings Microservice

**Purpose**: Manages application-wide configurations and user roles dynamically.

- **Endpoint**: `GET /api/v1/config/settings`
  - **Description**: Retrieves configuration settings for a specific service or the entire application.
  - **Parameters**:
    - `service_name` (string, required): The name of the service for which settings are requested (e.g., "authentication").
  - **Response**:
    - Success (200 OK): `{"service": "<service_name>", "settings": {"key": "value", ...}}`
    - Failure (404 Not Found): `{"status": "error", "error_code": "NOT_FOUND", "message": "Service settings not found"}`
    - Failure (401 Unauthorized): `{"status": "error", "error_code": "UNAUTHORIZED", "message": "Invalid or missing token"}`
  - **Security**: Requires JWT token in `Authorization` header for admin or service access.

- **Endpoint**: `POST /api/v1/config/update`
  - **Description**: Updates configuration settings for a specific service or application-wide.
  - **Parameters**:
    - `settings_data` (object, required): An object containing the updated settings (e.g., `{"service": "authentication", "settings": {"key": "new_value"}}`).
  - **Response**:
    - Success (200 OK): `{"status": "updated", "updated_settings": {"key": "new_value", ...}}`
    - Failure (400 Bad Request): `{"status": "error", "error_code": "INVALID_DATA", "message": "Invalid settings data provided"}`
    - Failure (401 Unauthorized): `{"status": "error", "error_code": "UNAUTHORIZED", "message": "Invalid or missing token"}`
  - **Security**: Requires JWT token in `Authorization` header for admin access only.

### 4. Logging System Microservice

**Purpose**: Collects and stores logs from all microservices for monitoring and debugging.

- **Endpoint**: `POST /api/v1/logs/submit`
  - **Description**: Submits log data from other microservices or the frontend for storage and processing.
  - **Parameters**:
    - `log_type` (string, required): The type of log (e.g., "user_activity", "debug", "terminal", "console").
    - `log_data` (object, required): An object containing log details (e.g., timestamp, message, source).
  - **Response**:
    - Success (201 Created): `{"status": "log_submitted", "log_id": "<log_id>"}`
    - Failure (400 Bad Request): `{"status": "error", "error_code": "INVALID_DATA", "message": "Invalid log data provided"}`
    - Failure (401 Unauthorized): `{"status": "error", "error_code": "UNAUTHORIZED", "message": "Invalid or missing token"}`
  - **Security**: Requires JWT token in `Authorization` header for service or admin access.

### 5. Secure Bot Data Storage Microservice

**Purpose**: Securely stores and retrieves sensitive bot data with encryption.

- **Endpoint**: `POST /api/v1/botdata/store`
  - **Description**: Stores sensitive bot data securely with encryption.
  - **Parameters**:
    - `bot_id` (string, required): The unique identifier of the bot.
    - `data` (object, required): An object containing sensitive data to be encrypted and stored.
  - **Response**:
    - Success (201 Created): `{"status": "stored", "bot_id": "<bot_id>"}`
    - Failure (400 Bad Request): `{"status": "error", "error_code": "INVALID_DATA", "message": "Invalid data provided"}`
    - Failure (401 Unauthorized): `{"status": "error", "error_code": "UNAUTHORIZED", "message": "Invalid or missing token"}`
  - **Security**: Requires JWT token in `Authorization` header for authorized access.

- **Endpoint**: `GET /api/v1/botdata/retrieve`
  - **Description**: Retrieves and decrypts sensitive bot data for a specific bot.
  - **Parameters**:
    - `bot_id` (string, required): The unique identifier of the bot whose data is to be retrieved.
  - **Response**:
    - Success (200 OK): `{"bot_id": "<bot_id>", "data": {"key": "decrypted_value", ...}}`
    - Failure (404 Not Found): `{"status": "error", "error_code": "NOT_FOUND", "message": "Bot data not found"}`
    - Failure (401 Unauthorized): `{"status": "error", "error_code": "UNAUTHORIZED", "message": "Invalid or missing token"}`
  - **Security**: Requires JWT token in `Authorization` header for authorized access.

## Future Considerations

- **API Expansion**: As additional microservices are developed in future phases (e.g., Content Engine, Trends Research), their API contracts will be appended to this document with consistent versioning and security practices.
- **Interactive Documentation**: Implement Swagger UI or similar tools for interactive API documentation and testing once the backend is fully operational.
- **Rate Limiting**: Plan for rate limiting at the API Gateway level to prevent abuse, which will be detailed in future infrastructure documentation.

This document serves as the authoritative reference for API contracts in Phase 1, ensuring that all microservices communicate effectively and securely as implementation progresses.