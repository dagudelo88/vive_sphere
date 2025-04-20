# API Gateway Setup for ViveSphere Bot Manager - Phase 1

**Objective**: Document the setup and configuration process for the API Gateway in Phase 1 of the ViveSphere Bot Manager system, ensuring centralized routing, authentication, rate limiting, and logging for all microservices.

## Overview

The API Gateway serves as the single entry point for all external requests to the ViveSphere Bot Manager system, handling routing, authentication, rate limiting, and logging for the microservices. In Phase 1, the focus is on establishing the foundational infrastructure for the API Gateway to support the core microservices (Authentication, Bot Manager, Configuration/Settings, Logging, and Secure Bot Data Storage). This document outlines the selection, configuration, and deployment process for the API Gateway.

## Selection of API Gateway

- **Chosen Gateway**: Kong
  - **Rationale**: Kong is selected for its robust feature set, including built-in plugins for authentication, rate limiting, and logging, as well as its compatibility with Kubernetes for future scalability. It also supports a wide range of customization options and has strong community and enterprise support, aligning with the scalability and future-proofing goals outlined in the PRD (section 3.1).
  - **Alternative**: Traefik was considered as an alternative due to its simplicity and native integration with Docker, but Kong was preferred for its more comprehensive feature set tailored to microservices architectures.

## Configuration

The API Gateway configuration is planned to be stored in `infrastructure/api_gateway/` directory. The configuration will define routing rules, authentication mechanisms, and rate limiting policies to manage traffic to the microservices.

- **Routing Rules**:
  - Map external requests to internal microservice endpoints based on path prefixes:
    - `/api/v1/auth/*` → Authentication Microservice (e.g., `http://authentication:8000/api/v1/auth/*`)
    - `/api/v1/bots/*` → Bot Manager Microservice (e.g., `http://bot_manager:8000/api/v1/bots/*`)
    - `/api/v1/config/*` → Configuration/Settings Microservice (e.g., `http://configuration:8000/api/v1/config/*`)
    - `/api/v1/logs/*` → Logging System Microservice (e.g., `http://logging:8000/api/v1/logs/*`)
    - `/api/v1/botdata/*` → Secure Bot Data Storage Microservice (e.g., `http://secure_bot_data:8000/api/v1/botdata/*`)
  - Ensure versioned API paths (e.g., `/api/v1/`) are preserved for backward compatibility as per PRD section 3.5.

- **Authentication**:
  - Implement a JWT authentication plugin to validate tokens for protected endpoints. Requests without valid tokens or with expired tokens will be rejected with a 401 Unauthorized response.
  - Public endpoints (e.g., `/api/v1/auth/login`) will be exempt from authentication checks to allow initial user access.
  - Configure the plugin to communicate with the Authentication Microservice for token validation if needed.

- **Rate Limiting**:
  - Set up rate limiting to prevent abuse and ensure fair usage of resources. Initial configuration will limit requests to 100 per minute per client IP for each microservice endpoint.
  - Rate limiting policies can be adjusted dynamically based on usage patterns and will be revisited in future phases for optimization.

- **Logging**:
  - Enable request logging to track incoming requests, responses, and errors. Logs will be forwarded to the Logging System Microservice via a configured plugin or direct API calls for centralized storage and analysis.

## Docker Setup

To ensure consistency across environments, the API Gateway will be deployed using Docker containers. The following setup files will be created in `infrastructure/api_gateway/`:

- **Dockerfile**: A basic Dockerfile for Kong, if customization is needed beyond the official image. Initially, the official Kong image will be used directly.
- **docker-compose.yml**: A configuration file for local development and testing, defining the Kong service with necessary environment variables and port mappings.

Example snippet for `docker-compose.yml`:
```yaml
services:
  api-gateway:
    image: kong:latest
    ports:
      - "8000:8000"
      - "8443:8443"
      - "8001:8001"
    environment:
      - KONG_DATABASE=off
      - KONG_PROXY_ACCESS_LOG=/dev/stdout
      - KONG_ADMIN_ACCESS_LOG=/dev/stdout
      - KONG_PROXY_ERROR_LOG=/dev/stderr
      - KONG_ADMIN_ERROR_LOG=/dev/stderr
      - KONG_ADMIN_LISTEN=0.0.0.0:8001
    volumes:
      - ./kong_config:/kong/declarative
    restart: always
```

- **Configuration Files**: Kong declarative configuration files (e.g., `kong.yml`) will be stored in `infrastructure/api_gateway/kong_config/` to define services, routes, and plugins without a database backend for simplicity in Phase 1.

## Deployment Steps

1. **Setup Docker Environment**:
   - Ensure Docker and Docker Compose are installed on the development environment.
   - Pull the latest Kong image using `docker pull kong:latest`.

2. **Configure Kong**:
   - Create or update the `docker-compose.yml` file in `infrastructure/api_gateway/` with the necessary settings.
   - Define initial routing rules, authentication plugins, and rate limiting policies in a declarative configuration file (`kong.yml`).

3. **Run Kong Locally**:
   - Start the Kong container using `docker-compose up -d api-gateway` from the `infrastructure/api_gateway/` directory.
   - Verify that Kong is running and accessible on port 8000 (proxy) and 8001 (admin API) using `curl` or a browser.

4. **Test Routing and Authentication**:
   - Test routing to mock or real microservice endpoints by sending requests to `http://localhost:8000/api/v1/auth/login` and confirming they reach the Authentication Microservice.
   - Test authentication by sending requests with and without JWT tokens to protected endpoints, ensuring appropriate responses (200 OK or 401 Unauthorized).

5. **Integrate with Microservices**:
   - Update Kong configuration to route to actual microservice instances once they are deployed locally or in a Kubernetes cluster.
   - Ensure microservices are accessible via their internal service names (e.g., `http://authentication:8000`) within the Docker network.

## Future Considerations

- **Kubernetes Integration**: In future phases, Kong will be deployed within a Kubernetes cluster using Helm charts or custom manifests for scalability and fault tolerance, as outlined in PRD section 3.1.
- **Advanced Plugins**: Additional Kong plugins for caching, request transformation, or advanced logging will be considered as the system grows and performance optimization becomes necessary.
- **Monitoring**: Integrate Kong with Prometheus and Grafana for monitoring API Gateway performance metrics, aligning with PRD section 3.6 on system monitoring.

This document provides the foundational setup plan for the API Gateway in Phase 1, ensuring centralized management of requests to the microservices as implementation progresses.