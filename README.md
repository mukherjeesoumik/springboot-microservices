# Spring Boot Microservices Demo

A comprehensive microservices architecture demonstration using Spring Boot, Spring Cloud Gateway, Netflix Eureka, and basic authentication.

## 🏗️ Architecture Overview

This project demonstrates a complete microservices setup with:
- **Service Discovery** using Netflix Eureka Server
- **API Gateway** with Spring Cloud Gateway
- **Load Balancing** with client-side load balancing
- **Security** with HTTP Basic Authentication
- **Multiple Services** (Service A & Service B)

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client        │───▶│   API Gateway   │───▶│  Eureka Server  │
│                 │    │   (Port 8080)   │    │   (Port 8761)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
            ┌───────▼────────┐  ┌──────▼─────────┐
            │   Service A     │  │   Service B    │
            │  (Port 8081)    │  │  (Port 8082)   │
            └─────────────────┘  └────────────────┘
```

## 📁 Project Structure

```
microservices-demo/
│
├── eureka-server/          # Service Discovery Server
│   ├── pom.xml
│   ├── src/main/java/com/example/eurekaserver/EurekaServerApplication.java
│   └── src/main/resources/application.yml
│
├── api-gateway/            # API Gateway with Security
│   ├── pom.xml
│   ├── src/main/java/com/example/apigateway/ApiGatewayApplication.java
│   ├── src/main/java/com/example/apigateway/config/SecurityConfig.java
│   └── src/main/resources/application.yml
│
├── service-a/              # Microservice A
│   ├── pom.xml
│   ├── src/main/java/com/example/servicea/ServiceAApplication.java
│   ├── src/main/java/com/example/servicea/controller/ServiceAController.java
│   └── src/main/resources/application.properties
│
└── service-b/              # Microservice B
    ├── pom.xml
    ├── src/main/java/com/example/serviceb/ServiceBApplication.java
    ├── src/main/java/com/example/serviceb/controller/ServiceBController.java
    └── src/main/resources/application.properties
```

## 🚀 Quick Start

### Prerequisites
- Java 17 or higher
- Maven 3.6+
- IDE (IntelliJ IDEA, Eclipse, or VS Code)

### Running the Application

**Important:** Start services in the following order:

1. **Start Eureka Server**
   ```bash
   cd eureka-server
   mvn spring-boot:run
   ```
   Access at: http://localhost:8761

2. **Start Service A**
   ```bash
   cd service-a
   mvn spring-boot:run
   ```
   Running on port: 8081

3. **Start Service B**
   ```bash
   cd service-b
   mvn spring-boot:run
   ```
   Running on port: 8082

4. **Start API Gateway**
   ```bash
   cd api-gateway
   mvn spring-boot:run
   ```
   Access at: http://localhost:8080

## 🔐 Authentication

The API Gateway is secured with HTTP Basic Authentication:
- **Username:** `admin`
- **Password:** `admin123`

## 🧪 Testing the Services

### Using cURL

**Test Service A:**
```bash
curl -u admin:admin123 http://localhost:8080/service-a/hello
```

**Test Service B:**
```bash
curl -u admin:admin123 http://localhost:8080/service-b/hello
```

### Using Browser
1. Navigate to http://localhost:8080/service-a/hello
2. Enter credentials when prompted
3. You should see: "Hello from Service A!"

## 📊 Service Endpoints

| Service | Direct URL | Gateway URL | Response |
|---------|------------|-------------|----------|
| Service A | http://localhost:8081/service-a/hello | http://localhost:8080/service-a/hello | "Hello from Service A!" |
| Service B | http://localhost:8082/service-b/hello | http://localhost:8080/service-b/hello | "Hello from Service B!" |
| Eureka Dashboard | http://localhost:8761 | - | Service Registry UI |

---

## 📝 Complete Source Code

### 1. Eureka Server

#### `pom.xml`
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.2</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>2023.0.3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

#### `src/main/java/com/example/eurekaserver/EurekaServerApplication.java`
```java
package com.example.eurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

#### `src/main/resources/application.yml`
```yaml
spring:
  application:
    name: eureka-server

server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

---

### 2. API Gateway

#### `pom.xml`
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>api-gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.2</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>2023.0.3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

#### `src/main/java/com/example/apigateway/ApiGatewayApplication.java`
```java
package com.example.apigateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

#### `src/main/java/com/example/apigateway/config/SecurityConfig.java`
```java
package com.example.apigateway.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        return http
                .csrf(ServerHttpSecurity.CsrfSpec::disable)
                .authorizeExchange(exchange -> exchange
                        .pathMatchers("/service-a/**", "/service-b/**").authenticated()
                        .anyExchange().permitAll()
                )
                .httpBasic()
                .and()
                .build();
    }
}
```

#### `src/main/resources/application.yml`
```yaml
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        - id: service-a
          uri: lb://service-a
          predicates:
            - Path=/service-a/**
        - id: service-b
          uri: lb://service-b
          predicates:
            - Path=/service-b/**
  security:
    user:
      name: admin
      password: admin123

server:
  port: 8080

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    prefer-ip-address: true
```

---

### 3. Service A

#### `pom.xml`
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>service-a</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.2</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>2023.0.3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

#### `src/main/java/com/example/servicea/ServiceAApplication.java`
```java
package com.example.servicea;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ServiceAApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceAApplication.class, args);
    }
}
```

#### `src/main/java/com/example/servicea/controller/ServiceAController.java`
```java
package com.example.servicea.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ServiceAController {
    @GetMapping("/service-a/hello")
    public String hello() {
        return "Hello from Service A!";
    }
}
```

#### `src/main/resources/application.properties`
```properties
spring.application.name=service-a
server.port=8081
eureka.client.service-url.defaultZone=http://localhost:8761/eureka
eureka.instance.prefer-ip-address=true
```

---

### 4. Service B

#### `pom.xml`
```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>service-b</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.2</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>2023.0.3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

#### `src/main/java/com/example/serviceb/ServiceBApplication.java`
```java
package com.example.serviceb;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ServiceBApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceBApplication.class, args);
    }
}
```

#### `src/main/java/com/example/serviceb/controller/ServiceBController.java`
```java
package com.example.serviceb.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ServiceBController {
    @GetMapping("/service-b/hello")
    public String hello() {
        return "Hello from Service B!";
    }
}
```

#### `src/main/resources/application.properties`
```properties
spring.application.name=service-b
server.port=8082
eureka.client.service-url.defaultZone=http://localhost:8761/eureka
eureka.instance.prefer-ip-address=true
```

---

## 🔧 Key Components

### 1. Eureka Server (Port 8761)
- **Purpose:** Service discovery and registration
- **Key Features:**
  - Central registry for all microservices
  - Health monitoring
  - Load balancing support

### 2. API Gateway (Port 8080)
- **Purpose:** Single entry point for all client requests
- **Key Features:**
  - Request routing to appropriate services
  - Load balancing with `lb://` URI scheme
  - HTTP Basic Authentication
  - CORS handling

### 3. Service A & B (Ports 8081, 8082)
- **Purpose:** Business logic microservices
- **Key Features:**
  - Auto-registration with Eureka
  - RESTful endpoints
  - Independent deployment

## ⚙️ Configuration Highlights

### Gateway Routes
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: service-a
          uri: lb://service-a    # Load balanced routing
          predicates:
            - Path=/service-a/**
        - id: service-b
          uri: lb://service-b
          predicates:
            - Path=/service-b/**
```

### Security Configuration
```yaml
spring:
  security:
    user:
      name: admin
      password: admin123
```

## 🛠️ Tech Stack

- **Spring Boot 3.3.2** - Application framework
- **Spring Cloud 2023.0.3** - Microservices toolkit
- **Spring Cloud Gateway** - API Gateway
- **Netflix Eureka** - Service discovery
- **Spring Security** - Authentication & authorization
- **Maven** - Build tool

## 📈 Monitoring

- **Eureka Dashboard:** http://localhost:8761
  - View registered services
  - Monitor service health
  - Check instance status

## 🔄 Load Balancing

The setup includes client-side load balancing:
- Multiple instances of the same service register with Eureka
- Gateway automatically distributes requests using round-robin
- Failed instances are automatically removed from load balancing

## 🚨 Troubleshooting

**Services not appearing in Eureka:**
- Ensure Eureka Server is running first
- Check `eureka.client.service-url.defaultZone` configuration
- Verify network connectivity

**Authentication failures:**
- Confirm username/password: `admin/admin123`
- Check if requests are going through the gateway (port 8080)

**Gateway routing issues:**
- Verify service names match in Eureka registry
- Check path predicates in gateway configuration

## 📝 Next Steps

This demo can be extended with:
- Docker containerization
- Database integration
- Circuit breaker patterns (Hystrix/Resilience4j)
- Distributed tracing (Sleuth/Zipkin)
- Configuration management (Spring Cloud Config)
- Message queues (RabbitMQ/Kafka)

## 🤝 Contributing

Feel free to fork this project and submit pull requests for improvements!

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
