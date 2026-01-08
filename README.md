# Cloud Gateway Service - StudyBuddy

The **Cloud Gateway** serves as the single entry point for the StudyBuddy microservices architecture. It provides dynamic routing, monitoring, and security (authentication & authorization) for backend services.

## 🚀 Features

*   **Centralized Routing**: Intelligently routes API requests to the appropriate microservices.
*   **JWT Authentication**: Validates JSON Web Tokens (JWT) for protected routes using a custom Gateway Filter.
*   **Header Injection**: Extracts user details (ID, Roles) from the JWT and injects them as headers (`X-User-Id`, `X-User-Roles`) for downstream services.
*   **Health Monitoring**: Integrated Spring Boot Actuator for health checks and metrics.
*   **CORS Support**: Handles Cross-Origin Resource Sharing preflight requests.

## 🛠️ Tech Stack

*   **Language**: Java 25
*   **Framework**: Spring Boot 3.5.9
*   **Gateway**: Spring Cloud Gateway (2023.0.3)
*   **Security**: Spring Security, JJWT (0.11.5)
*   **Build Tool**: Gradle

## ⚙️ Configuration

### Application Properties (`application.yaml`)

The gateway runs on port **8080** by default.

| Property | Description | Default Value |
| :--- | :--- | :--- |
| `server.port` | Port where the gateway runs | `8080` |
| `app.jwtSecret` | Secret key for signing/validating JWTs | *Configured in yaml* |
| `management.endpoints...` | Actuator endpoints exposed | `health,info,metrics` |

### API Routes

The gateway is configured to route traffic to the following services:

| Service ID | Upstream URL | Path Predicates |
| :--- | :--- | :--- |
| **user_identity_service** | `http://localhost:8081` | `/api/v1/auth/**`, `/api/v1/users/**` |
| **collaboration_service** | `http://localhost:8082` | `/api/v1/groups/**`, `/api/v1/files/**` |

> **Note**: Ensure the upstream services are running on their respective ports locally.

## 🔒 Security Flow

The `JwtAuthGatewayFilter` handles request security:

1.  **Public Endpoints**: Requests to `/api/v1/auth` are whitelisted and forwarded immediately.
2.  **CORS**: `OPTIONS` requests are allowed to pass through for browser preflight checks.
3.  **Token Validation**: All other requests must include an `Authorization` header with a valid Bearer token.
4.  **Claims Extraction**: The filter validates the token signature using `app.jwtSecret`.
5.  **Context Propagation**:
    *   `X-User-Id`: Extracted from the token subject.
    *   `X-User-Roles`: Extracted from the `roles` claim.
    *   These headers are added to the request before it is sent to the downstream service.
6.  **Unauthorized**: If the token is missing or invalid, a `401 Unauthorized` response is returned.

## 📦 Getting Started

### Prerequisites
*   Java 25 SDK installed.
*   Gradle installed (or use the provided wrapper).
*   Docker (optional)

### Build and Run

1.  **Clone the repository**
2.  **Build the project**:
    ```bash
    ./gradlew clean build
    ```
3.  **Run the application**:
    ```bash
    ./gradlew bootRun
    ```

### Docker Support

To build and run the application using Docker:

1.  **Build the Docker image**:
    ```bash
    docker build -t cloud-gateway .
    ```
2.  **Run the container**:
    ```bash
    docker run -p 8080:8080 cloud-gateway
    ```

### Testing

Once running, you can access the actuator health endpoint:
```
GET http://localhost:8080/actuator/health
```

To test routing, send a request to a configured path (e.g., `http://localhost:8080/api/v1/auth/login`) and ensure the target service is running.
