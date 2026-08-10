# Service Discovery

A Spring Boot–based **Service Discovery Server** built using **Netflix Eureka and Spring Cloud**.

This service acts as the central service registry for a microservices architecture. Individual microservices such as `UserAuthService`, `ProductCatalogService`, and other backend services can register themselves with Eureka and discover other services by logical service name instead of relying on hard-coded hostnames and ports.

---

# 🚀 Overview

In a traditional architecture, one service might call another using a fixed URL:

```text
http://localhost:8081/users/1
```

This becomes problematic when:

* Service instances move between hosts
* Multiple instances of a service are running
* Ports change
* Services scale horizontally
* Containers are dynamically created/destroyed

Service discovery solves this by introducing a **service registry**.

Instead of calling a fixed host:

```text
ProductCatalogService
        │
        │ "Where is UserAuthService?"
        ▼
   Eureka Server
        │
        │ returns available instances
        ▼
UserAuthService instances
```

The current application provides the Eureka server that acts as this registry. Its configuration disables both registry fetching and self-registration, which is appropriate for a standalone Eureka server.

---

# 🏗️ Architecture

The service discovery architecture can be visualized as:

```text
                         ┌──────────────────────┐
                         │    Eureka Server     │
                         │                      │
                         │       :8761          │
                         │                      │
                         │   Service Registry   │
                         └──────────┬───────────┘
                                    │
                    ┌───────────────┼────────────────┐
                    │               │                │
              Register          Register         Register
                    │               │                │
                    ▼               ▼                ▼
          ┌────────────────┐ ┌───────────────┐ ┌───────────────┐
          │ UserAuthService│ │ProductCatalog │ │ PaymentService│
          │                │ │   Service     │ │               │
          └────────────────┘ └───────────────┘ └───────────────┘
```

A service can register itself with Eureka and subsequently be discovered using its logical application name.

Netflix describes Eureka as a REST-based service registry used for discovery, load balancing, and failover.

---

# 🔍 Why Service Discovery?

Without service discovery:

```text
ProductCatalogService
        │
        │ Hard-coded URL
        ▼
http://localhost:8081/users/1
```

With service discovery:

```text
ProductCatalogService
        │
        │ Discover "UserAuthService"
        ▼
   Eureka Server
        │
        ▼
UserAuthService
```

The client no longer needs to know the physical location of the target service.

---

# 🧠 Core Concept

The Eureka server maintains information about available service instances.

Conceptually:

```text
Service Registry
│
├── UserAuthService
│      ├── Instance 1 → host:port
│      └── Instance 2 → host:port
│
├── ProductCatalogService
│      └── Instance 1 → host:port
│
└── PaymentService
       ├── Instance 1 → host:port
       └── Instance 2 → host:port
```

When a service needs another service, it can query the registry instead of maintaining a static list of server addresses.

---

# 🛠️ Tech Stack

| Technology                         | Purpose                         |
| ---------------------------------- | ------------------------------- |
| Java 17                            | Programming language            |
| Spring Boot 3.5.3                  | Application framework           |
| Spring Web                         | Web/application infrastructure  |
| Spring Cloud Netflix Eureka Server | Service registry                |
| Maven                              | Build and dependency management |
| Lombok                             | Boilerplate reduction           |
| Spring Boot Test                   | Testing                         |

The repository uses Java 17, Spring Boot `3.5.3`, and `spring-cloud-starter-netflix-eureka-server` version `4.3.1`.

---

# 📁 Project Structure

```text
ServiceDiscovery/
│
├── .mvn/
│   └── wrapper/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── org/example/servicediscovery/
│   │   │       └── ServiceDiscoveryApplication.java
│   │   │
│   │   └── resources/
│   │       └── application.properties
│   │
│   └── test/
│       └── java/
│           └── org/example/servicediscovery/
│
├── pom.xml
├── mvnw
├── mvnw.cmd
├── .gitignore
└── .gitattributes
```

The project is intentionally small because the application primarily provides infrastructure rather than business functionality.

---

# ⚙️ Application Configuration

The current configuration is:

```properties
spring.application.name=ServiceDiscovery

server.port=8761

eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
```

### Application name

```properties
spring.application.name=ServiceDiscovery
```

Identifies the Eureka server application.

### Server port

```properties
server.port=8761
```

Eureka conventionally exposes its dashboard and registry endpoints on port `8761`.

### Fetch registry

```properties
eureka.client.fetch-registry=false
```

The Eureka server does not need to fetch service information from another registry.

### Register with Eureka

```properties
eureka.client.register-with-eureka=false
```

The Eureka server does not register itself as a client.

This configuration is appropriate for a standalone Eureka server.

---

# ▶️ Running the Service

## Prerequisites

Install:

* Java 17+
* Maven 3.8+
* Git

---

## 1. Clone the repository

```bash
git clone https://github.com/vikramhankare/ServiceDiscovery.git
```

```bash
cd ServiceDiscovery
```

---

## 2. Build the project

Using Maven:

```bash
mvn clean install
```

Or the Maven wrapper:

### Linux / macOS

```bash
./mvnw clean install
```

### Windows

```cmd
mvnw.cmd clean install
```

---

## 3. Start Eureka Server

```bash
mvn spring-boot:run
```

Or:

```bash
./mvnw spring-boot:run
```

Windows:

```cmd
mvnw.cmd spring-boot:run
```

The Eureka server starts on:

```text
http://localhost:8761
```

---

# 🌐 Eureka Dashboard

Once the application is running, open:

```text
http://localhost:8761
```

The Eureka dashboard provides a visual representation of registered service instances.

Conceptually:

```text
┌─────────────────────────────────────────────┐
│              Eureka Dashboard               │
├─────────────────────────────────────────────┤
│                                             │
│  Registered Instances                       │
│                                             │
│  USERAUTHSERVICE                            │
│     └── localhost:8080                      │
│                                             │
│  PRODUCTCATALOGSERVICE                      │
│     └── localhost:8081                      │
│                                             │
│  PAYMENTSERVICE                             │
│     └── localhost:8082                      │
│                                             │
└─────────────────────────────────────────────┘
```

The actual services shown depend on which Eureka clients are currently running.

---

# 🔗 Registering a Microservice

A Spring Boot service that wants to register with this Eureka server needs the Eureka Client dependency.

For example:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

Then configure:

```properties
spring.application.name=UserAuthService

eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true

eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
```

The service can then register itself with the Eureka server.

---

# 🔄 Service Registration Flow

```text
                    Eureka Server
                    localhost:8761
                          ▲
                          │
                 Register Service
                          │
                          │
                ┌─────────┴─────────┐
                │                   │
                │                   │
       UserAuthService      ProductCatalogService
          :8080                   :8081
```

When the services start:

```text
UserAuthService
      │
      │ REGISTER
      ▼
Eureka Server
```

and:

```text
ProductCatalogService
      │
      │ REGISTER
      ▼
Eureka Server
```

Eureka maintains the registry of available instances.

---

# 🔎 Service Discovery Flow

Suppose `ProductCatalogService` needs to communicate with `UserAuthService`.

Without Eureka:

```text
ProductCatalogService
        │
        ▼
http://localhost:8080/users/1
```

With Eureka:

```text
ProductCatalogService
        │
        │ Discover UserAuthService
        ▼
   Eureka Server
        │
        │ Available instance
        ▼
UserAuthService
```

The calling service can use the logical service name rather than a physical hostname.

---

# ⚖️ Multiple Service Instances

One of the major advantages of service discovery is supporting multiple instances.

For example:

```text
                    Eureka
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       UserAuth    UserAuth    UserAuth
       :8080       :8081       :8082
```

The registry knows that multiple instances of the same logical service are available.

A client-side load-balancing mechanism can then select an instance.

This is particularly useful when scaling microservices horizontally.

---

# 🧩 Relationship with the Other Services

This repository acts as the infrastructure component connecting your other microservices.

A simplified architecture for your current project set is:

```text
                         ┌───────────────────┐
                         │   ServiceDiscovery│
                         │      Eureka       │
                         │       :8761       │
                         └─────────┬─────────┘
                                   │
             ┌─────────────────────┼─────────────────────┐
             │                     │                     │
             ▼                     ▼                     ▼
      ┌───────────────┐    ┌─────────────────┐    ┌───────────────┐
      │ UserAuth      │    │ ProductCatalog  │    │   Payment     │
      │ Service       │    │ Service         │    │   Service     │
      └───────┬───────┘    └─────────────────┘    └───────────────┘
              │
              │ Signup Event
              ▼
         ┌──────────┐
         │  Kafka   │
         └────┬─────┘
              │
              ▼
      ┌─────────────────┐
      │ Notification    │
      │ Service         │
      └─────────────────┘
```

This makes the repositories more meaningful as a collection:

```text
ServiceDiscovery
      │
      ├── UserAuthService
      │
      ├── ProductCatalogService
      │
      ├── PaymentService
      │
      └── NotificationService
```

The Eureka server provides **service registration and discovery**, while Kafka provides **asynchronous event communication** between services.

---

# 🧠 Service Discovery vs Direct Communication

## Without Service Discovery

```text
ProductCatalogService
       │
       │ hard-coded IP/port
       ▼
UserAuthService
```

Problems:

* Tight coupling to network location
* Difficult scaling
* Configuration changes when ports/hosts change
* Difficult dynamic instance management

---

## With Service Discovery

```text
ProductCatalogService
       │
       │ service name
       ▼
Eureka
       │
       ▼
UserAuthService instance
```

Benefits:

* Logical service names
* Dynamic service registration
* Easier horizontal scaling
* Reduced dependency on fixed hostnames
* Centralized service registry

---

# 🏛️ Client-Side Discovery

The architecture used by Eureka commonly follows the **client-side discovery** model.

```text
                ┌──────────────┐
                │    Eureka    │
                │   Registry   │
                └──────┬───────┘
                       │
                  Service List
                       │
                       ▼
              ┌─────────────────┐
              │ Service Client  │
              └────────┬────────┘
                       │
                Select Instance
                       │
                       ▼
              Target Microservice
```

The client obtains information about available service instances from the registry.

This differs from server-side discovery, where a load balancer or intermediary performs the discovery and routing.

---

# 🛠️ Useful Commands

## Build

```bash
mvn clean install
```

## Run

```bash
mvn spring-boot:run
```

## Run tests

```bash
mvn test
```

## Package JAR

```bash
mvn clean package
```

## Run generated JAR

```bash
java -jar target/ServiceDiscovery-0.0.1-SNAPSHOT.jar
```

---

# 🧪 Testing

The project includes Spring Boot's test dependency and a test source directory.

Run:

```bash
mvn test
```

For a service-discovery server, useful tests can include:

* Application context startup
* Eureka server availability
* Client registration
* Service lookup
* Multiple instance registration
* Service deregistration behavior

---

# 🔐 Production Considerations

The current repository is a minimal learning implementation of a Eureka server.

For production use, additional considerations include:

### High Availability

A single Eureka server introduces a potential infrastructure dependency.

A production setup can use multiple Eureka server instances:

```text
             ┌───────────────┐
             │ Eureka Server 1│
             └───────┬───────┘
                     │
             ┌───────┴───────┐
             │ Eureka Server 2│
             └───────────────┘
```

### Security

The registry should not necessarily be publicly accessible.

Consider:

* Network isolation
* Authentication
* Authorization
* TLS
* Secure service-to-service communication

### Health Checks

Service instances should expose reliable health information so unavailable instances can be removed from active routing.

### Observability

Consider:

* Metrics
* Logging
* Distributed tracing
* Eureka registry monitoring
* Spring Boot Actuator

---

# ⚠️ Current Scope & Limitations

This repository currently provides a minimal Eureka server.

The current application contains:

* Spring Boot application entry point
* Eureka Server dependency
* Eureka server configuration
* Java 17 configuration
* Spring Boot test setup

It does **not** currently contain:

* Custom REST APIs
* Database
* Authentication
* Authorization
* Docker configuration
* Docker Compose
* CI/CD configuration
* Custom health-check endpoints
* High-availability Eureka cluster configuration

The repository currently has a single commit and a very small codebase, which is appropriate for an infrastructure-learning project focused specifically on service discovery.

---

# 📈 Recommended Improvements

If you want to turn this into a stronger microservices infrastructure project:

* [ ] Add Eureka high-availability configuration
* [ ] Add Dockerfile
* [ ] Add Docker Compose
* [ ] Run Eureka alongside your other services
* [ ] Add Spring Boot Actuator
* [ ] Add health monitoring
* [ ] Add Eureka authentication
* [ ] Add HTTPS/TLS
* [ ] Add service metadata
* [ ] Document client-side load balancing
* [ ] Add integration tests
* [ ] Add GitHub Actions
* [ ] Add architecture diagrams
* [ ] Add centralized configuration using Spring Cloud Config

---

# 🎯 Learning Objectives

This project demonstrates the fundamentals of:

* Microservice architecture
* Service discovery
* Service registration
* Eureka
* Spring Cloud Netflix
* Client-side discovery
* Dynamic service locations
* Horizontal scaling concepts
* Microservice infrastructure

The key concept is:

```text
                    Service Discovery

       "I know WHAT service I need,
        but not WHERE it is running."

                         │
                         ▼

                    Eureka Server
                         │
                         ▼

                 Available Instances
```

---

# 👨‍💻 Author

**Vikram Hankare**

GitHub: https://github.com/vikramhankare

LinkedIn: https://www.linkedin.com/in/vikramhankare19