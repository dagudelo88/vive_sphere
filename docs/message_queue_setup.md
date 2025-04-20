# Message Queue Setup for ViveSphere Bot Manager - Phase 1

**Objective**: Document the setup and configuration process for the Message Queue infrastructure in Phase 1 of the ViveSphere Bot Manager system, enabling asynchronous communication between microservices for tasks such as logging and bot actions.

## Overview

The Message Queue infrastructure is critical for decoupling microservices in the ViveSphere Bot Manager system, allowing asynchronous communication for non-immediate tasks like logging, content generation triggers, or bot action scheduling. In Phase 1, the focus is on establishing the foundational setup for the Message Queue to support the core microservices. This document outlines the selection, configuration, and deployment process for the Message Queue system.

## Selection of Message Queue

- **Chosen Queue**: RabbitMQ
  - **Rationale**: RabbitMQ is selected for its reliability, ease of use, and robust support for various messaging patterns (e.g., publish/subscribe, work queues). It is well-suited for the asynchronous communication needs outlined in the PRD (section 3.1), such as logging and bot action processing, and integrates seamlessly with Python microservices using libraries like `pika`. Additionally, RabbitMQ offers a management UI for monitoring queues, which aids in debugging during development.
  - **Alternative**: Kafka was considered as an alternative due to its high throughput and scalability for event streaming, but RabbitMQ was preferred for its simplicity and suitability for the current scale of Phase 1 requirements.

## Configuration

The Message Queue configuration is planned to be stored in the `infrastructure/message_queue/` directory. The configuration will define specific queues for different tasks, connection parameters, and security settings to manage asynchronous communication between microservices.

- **Defined Queues**:
  - `log_queue`: For collecting logs from all microservices to be processed by the Logging System Microservice. This queue will handle high-frequency messages with varying priorities (e.g., INFO, DEBUG, ERROR).
  - `bot_action_queue`: For scheduling and processing bot actions triggered by the Bot Manager Microservice. This queue will manage tasks like posting content or interacting with the X platform API asynchronously.

- **Connection Parameters**:
  - **Host**: `rabbitmq` (service name within Docker network or Kubernetes cluster).
  - **Port**: `5672` (default RabbitMQ port for AMQP protocol).
  - **Credentials**: Use environment variables for username and password to prevent exposure in configuration files or version control. Default placeholders will be `guest`/`guest` for development, to be overridden in production.
  - **Virtual Host**: `/` (default virtual host for simplicity in Phase 1).

- **Queue Properties**:
  - **Durability**: Queues will be set as durable to survive broker restarts.
  - **Message TTL**: Configure a default Time-To-Live (TTL) of 7 days for messages in queues to manage storage, adjustable based on retention needs.
  - **Dead Letter Exchange**: Plan for a dead letter exchange (`dlx`) for handling failed messages, routing them to a separate queue (`dead_letter_queue`) for analysis.

## Docker Setup

To ensure consistency across environments, RabbitMQ will be deployed using Docker containers. The following setup files will be created in `infrastructure/message_queue/`:

- **Dockerfile**: A basic Dockerfile for RabbitMQ, if customization is needed beyond the official image. Initially, the official RabbitMQ image with management UI will be used directly.
- **docker-compose.yml**: A configuration file for local development and testing, defining the RabbitMQ service with necessary environment variables and port mappings.

Example snippet for `docker-compose.yml`:
```yaml
services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"  # AMQP port for client connections
      - "15672:15672"  # Management UI port
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest
      - RABBITMQ_DEFAULT_VHOST=/
    volumes:
      - ./rabbitmq_data:/var/lib/rabbitmq
    restart: always
```

- **Data Persistence**: Use a volume (`rabbitmq_data`) to persist queue data and configurations across container restarts, ensuring no loss of messages during development or testing.

## Integration with Microservices

- **Producer Services**: Microservices like Authentication, Bot Manager, and others will act as producers, publishing messages to specific queues (e.g., `log_queue` for logs, `bot_action_queue` for bot tasks) using a Python library like `pika` or `aio-pika` for asynchronous operations.
  - **Example**: The Logging System Microservice will consume from `log_queue`, while the Bot Manager Microservice will publish to `bot_action_queue` for scheduled tasks.
- **Consumer Services**: Dedicated microservices or background workers will consume messages from queues to process tasks asynchronously.
  - **Example**: A worker in the Bot Manager Microservice will consume from `bot_action_queue` to execute bot actions without blocking the main API endpoint.
- **Connection Management**: Implement connection pooling and retry mechanisms in microservices to handle temporary RabbitMQ unavailability, ensuring robustness as per PRD section 3.6.

## Deployment Steps

1. **Setup Docker Environment**:
   - Ensure Docker and Docker Compose are installed on the development environment.
   - Pull the latest RabbitMQ image using `docker pull rabbitmq:3-management`.

2. **Configure RabbitMQ**:
   - Create or update the `docker-compose.yml` file in `infrastructure/message_queue/` with the necessary settings for ports, environment variables, and volumes.
   - Define initial queues, exchanges, and bindings if needed via a configuration file or through the management UI post-deployment.

3. **Run RabbitMQ Locally**:
   - Start the RabbitMQ container using `docker-compose up -d rabbitmq` from the `infrastructure/message_queue/` directory.
   - Verify that RabbitMQ is running and accessible on port 5672 (AMQP) and 15672 (management UI) using `curl` or by accessing `http://localhost:15672` in a browser with default credentials (`guest`/`guest`).

4. **Test Queue Operations**:
   - Test basic queue operations by publishing test messages to `log_queue` and `bot_action_queue` using a simple Python script with `pika`.
   - Confirm messages are received by setting up a temporary consumer script or checking the management UI for queue status and message counts.

5. **Integrate with Microservices**:
   - Update microservice configurations to connect to RabbitMQ using the defined host, port, and credentials.
   - Implement producer and consumer logic in relevant microservices to publish and consume messages from the defined queues.

## Future Considerations

- **Kubernetes Integration**: In future phases, RabbitMQ will be deployed within a Kubernetes cluster using Helm charts or custom manifests for high availability and scalability, aligning with PRD section 3.1.
- **High Availability**: Plan for RabbitMQ clustering in production to ensure fault tolerance and data replication across nodes.
- **Monitoring**: Integrate RabbitMQ with Prometheus and Grafana for monitoring queue metrics (e.g., message rates, queue lengths), as outlined in PRD section 3.6.
- **Security**: Enhance security in production by enforcing TLS for connections and using non-default credentials managed via secret management tools (e.g., HashiCorp Vault), per PRD section 3.7.

This document provides the foundational setup plan for the Message Queue infrastructure in Phase 1, ensuring asynchronous communication capabilities as implementation progresses.