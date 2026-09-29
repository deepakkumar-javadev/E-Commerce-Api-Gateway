# E-Commerce API Gateway

API Gateway service for an e-commerce application built using Spring Boot and Spring Cloud.

The gateway provides a single entry point for the frontend application and routes requests to the required backend microservices. It also integrates with Eureka Service Discovery so that services can be accessed by their registered service names instead of fixed host and port combinations.

## Architecture

```text
                    Frontend
                       |
                       | HTTP
                       v
              +-------------------+
              |   API Gateway     |
              |      :8080        |
              +---------+---------+
                        |
                        | Service Discovery
                        v
                 +-------------+
                 |    Eureka   |
                 |    :8761    |
                 +-------------+
                        |
        +---------------+----------------+
        |               |                |
        v               v                v
   User Service    Product Service   Inventory Service
      :8082             :8083             :8084

        |               |                |
        +---------------+----------------+
                        |
        +---------------+----------------+
        |               |                |
        v               v                v
   Cart Service     Order Service     Payment Service
      :8085             :8086             :8087
                                            |
                                            v
                                    Notification Service
                                            :8088
```

## What this service does

The API Gateway is responsible for:

* Providing a single entry point for frontend requests
* Routing requests to different microservices
* Using Eureka for service discovery
* Handling CORS configuration for the frontend
* Forwarding HTTP requests to backend services
* Hiding individual service ports from the frontend
* Making the overall microservices setup easier to manage

Instead of the frontend calling every service separately, requests are sent through the gateway.

Example:

```text
Frontend
   |
   | GET /product/searchproducts
   v
API Gateway :8080
   |
   v
Product Service :8083
```

## Services

The application contains the following backend services:

| Service              | Port | Responsibility                                |
| -------------------- | ---: | --------------------------------------------- |
| User Service         | 8082 | User registration, login and user information |
| Product Service      | 8083 | Product management and product search         |
| Inventory Service    | 8084 | Stock management                              |
| Cart Service         | 8085 | Shopping cart operations                      |
| Order Service        | 8086 | Order creation and order management           |
| Payment Service      | 8087 | Payment processing                            |
| Notification Service | 8088 | Order/payment related notifications           |
| Eureka Server        | 8761 | Service discovery                             |
| API Gateway          | 8080 | Request routing and entry point               |

## Gateway Routes

The frontend accesses backend services through gateway routes.

Typical routes include:

```text
/auth/**           -> User Service
/product/**        -> Product Service
/stock/**          -> Inventory Service
/cart/**           -> Cart Service
/orders/**         -> Order Service
/payments/**       -> Payment Service
/notifications/**  -> Notification Service
```

For example:

```text
http://localhost:8080/product/searchproducts
```

The gateway receives the request and forwards it to the Product Service.

## Service Discovery

Eureka is used for service registration and discovery.

Each microservice registers itself with the Eureka Server. The API Gateway can then locate services using their registered service names.

Basic flow:

```text
Service starts
     |
     v
Registers with Eureka
     |
     v
API Gateway receives request
     |
     v
Gateway finds required service
     |
     v
Request is forwarded
```

This avoids hard-coding every service's host and port in the gateway configuration.

## Technology Stack

* Java 17
* Spring Boot 3.5.6
* Spring Cloud
* Spring Cloud Gateway
* Spring Cloud Netflix Eureka Client
* Spring Boot Actuator
* Maven
* REST APIs

## Project Structure

```text
01_Ecom-Api-Gateway
|
├── .mvn
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── deepak
│   │   │           └── gateway
│   │   │               └── Application.java
│   │   │
│   │   └── resources
│   │       └── application.yml
│   │
│   └── test
│       └── java
│
├── .gitignore
├── .gitattributes
├── mvnw
├── mvnw.cmd
├── pom.xml
└── README.md
```

## Configuration

The main gateway configuration is maintained in:

```text
src/main/resources/application.yml
```

The configuration contains settings for:

* Application name
* Gateway port
* Eureka connection
* Gateway routes
* CORS
* Actuator endpoints

Example application configuration:

```yaml
spring:
  application:
    name: ECOM-API-GATEWAY

server:
  port: 8080
```

The actual route configuration is maintained in the project's `application.yml`.

## CORS

The frontend application runs separately from the backend services, so CORS configuration is required.

Current frontend:

```text
http://localhost:9090
```

The gateway allows the frontend to communicate with backend services through the gateway.

## Actuator

Spring Boot Actuator is included for application monitoring and health information.

Example endpoint:

```text
http://localhost:8080/actuator/health
```

This can be used to check whether the gateway application is running.

## Running the Application

### Prerequisites

Make sure the following are installed:

* Java 17
* Maven
* MySQL
* Eureka Server
* Required backend microservices

### Start Eureka Server

Start the Eureka Server first:

```text
http://localhost:8761
```

### Start Backend Services

Start the required services:

```text
User Service       :8082
Product Service    :8083
Inventory Service  :8084
Cart Service       :8085
Order Service      :8086
Payment Service    :8087
Notification       :8088
```

### Start API Gateway

Using Maven Wrapper:

Windows:

```bash
mvnw.cmd spring-boot:run
```

Or using Maven:

```bash
mvn spring-boot:run
```

The gateway will start on:

```text
http://localhost:8080
```

## Request Flow

A typical product request works like this:

```text
Browser
   |
   | GET /product/searchproducts
   |
   v
API Gateway :8080
   |
   | Service Discovery
   v
Product Service :8083
   |
   v
Product Database
```

For an order-related request:

```text
Frontend
   |
   v
API Gateway
   |
   v
Order Service
   |
   +----> Cart Service
   |
   +----> Inventory Service
   |
   +----> Payment Service
   |
   +----> Notification Service
```

The gateway mainly handles the entry point and routing. Business logic remains inside the respective microservices.

## Why API Gateway is used

Without a gateway, the frontend would need to know the address and port of every backend service.

For example:

```text
Frontend
   |
   +--> User :8082
   +--> Product :8083
   +--> Cart :8085
   +--> Order :8086
   +--> Payment :8087
```

With the gateway:

```text
Frontend
     |
     v
API Gateway :8080
     |
     +--> User Service
     +--> Product Service
     +--> Cart Service
     +--> Order Service
     +--> Payment Service
```

This gives the frontend a single backend entry point and keeps individual service details behind the gateway.

## Microservices Communication

The overall application uses different communication approaches depending on the requirement.

### Synchronous communication

REST APIs and OpenFeign are used where an immediate response is required between services.

Example:

```text
Order Service
      |
      | REST / Feign
      v
Cart / Inventory Service
```

### Asynchronous communication

Apache Kafka is used for event-based communication where services do not need to wait for an immediate response.

Example:

```text
Order Service
      |
      | Kafka Event
      v
Inventory / Notification / Payment related consumers
```

The API Gateway itself is primarily responsible for HTTP request routing.

## Authentication

Authentication is handled by the User Service using JWT-based security.

The frontend sends the JWT token with authenticated requests:

```text
Authorization: Bearer <token>
```

The gateway forwards requests to the appropriate backend service according to the configured routes.

## Error Handling

If a requested backend service is unavailable, the gateway cannot successfully forward the request.

For example:

```text
Frontend
   |
   v
API Gateway
   |
   X
Product Service unavailable
```

This makes service availability easier to identify during development and testing.

## API Access

Once the gateway is running, APIs can be accessed using:

```text
http://localhost:8080
```

Examples:

```text
GET  http://localhost:8080/product/searchproducts

GET  http://localhost:8080/stock/getInventories

GET  http://localhost:8080/cart/getcart/{userId}

GET  http://localhost:8080/orders/getorder/{orderId}
```

The exact endpoints depend on the controllers and route configuration of each service.

## Related Project

This API Gateway is part of an e-commerce microservices application containing separate services for users, products, inventory, cart, orders, payments and notifications.

The gateway provides the common entry point for these services while Eureka handles service discovery.

## Technology Used: 

Technologies: Java, Spring Boot, Spring Cloud, Microservices, REST APIs, MySQL, Kafka
