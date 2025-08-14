# Spring Boot Microservices Demo

A simple microservices architecture using Spring Boot, Spring Cloud Gateway, and Netflix Eureka for service discovery.

## 🏗️ Architecture Overview

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │──▶│ API Gateway │───▶│   Eureka    │───▶│  Services   │
│  (Browser)  │    │ Port: 8080  │    │ Port: 8761  │    │             │
│             │    │             │    │             │    │             │
│   Request   │    │┌───────────┐│    │┌───────────┐│    │┌───────────┐│
│     ───▶    │    ││Load       ││    ││ Service   ││    ││Service A  ││
│             │    ││Balancer   ││    ││ Registry  ││    ││Port: 8081 ││
│   Response  │    │└───────────┘│    │└───────────┘│    │└───────────┘│
│    ◀───     │    │             │    │             │    │             │
└─────────────┘    │┌───────────┐│    │ ┌─────────┐ │    │┌───────────┐│
                   ││Route      ││    │ │service-a│ │    ││Service B  ││
                   ││Handler    ││    │ │service-b│ │    ││Port: 8082 ││
                   │└───────────┘│    │ └─────────┘ │    │└───────────┘│
                   └─────────────┘    └─────────────┘    └─────────────┘

🔄 Horizontal Request Flow (Left to Right):

Client Request ───▶ API Gateway ───▶ Service Discovery ───▶ Service Selection ───▶ Response

┌─────────────────────────────────────────────────────────────────────────────────────────┐
│  1. HTTP Request    2. Load Balance    3. Service Lookup    4. Route to Service         │
│  Client ──▶ Gateway ──▶ Eureka Lookup ──▶ Service A/B ──▶ Response ──▶ Client         │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

## 📂 Project Folder Structure
```
microservices-demo/
│
├── eureka-server/
│   ├── src/main/java/com/example/eurekaserver/EurekaServerApplication.java
│   └── src/main/resources/application.yml
│   └── pom.xml
│
├── api-gateway/
│   ├── src/main/java/com/example/apigateway/ApiGatewayApplication.java
│   └── src/main/resources/application.yml
│   └── pom.xml
│
├── service-a/
│   ├── src/main/java/com/example/servicea/ServiceAApplication.java
│   ├── src/main/java/com/example/servicea/controller/ServiceAController.java
│   └── src/main/resources/application.properties
│   └── pom.xml
│
├── service-b/
│   ├── src/main/java/com/example/serviceb/ServiceBApplication.java
│   ├── src/main/java/com/example/serviceb/controller/ServiceBController.java
│   └── src/main/resources/application.properties
│   └── pom.xml
│
└── pom.xml   (optional parent pom if using multi-module)
```

---

## 1️⃣ Eureka Server

### pom.xml
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```

### EurekaServerApplication.java
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

### application.yml
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

## 2️⃣ API Gateway

### pom.xml
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

### ApiGatewayApplication.java
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

### application.yml
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
server:
  port: 8080

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

---

## 3️⃣ Service A

### pom.xml
```xml
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
```

### ServiceAApplication.java
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

### ServiceAController.java
```java
package com.example.servicea.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ServiceAController {

    @GetMapping("/service-a/hello")
    public String hello() {
        return "Hello from Service A";
    }
}
```

### application.properties
```properties
spring.application.name=service-a
server.port=8081
eureka.client.service-url.defaultZone=http://localhost:8761/eureka
```

---

## 4️⃣ Service B

### pom.xml
```xml
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
```

### ServiceBApplication.java
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

### ServiceBController.java
```java
package com.example.serviceb.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ServiceBController {

    @GetMapping("/service-b/hello")
    public String hello() {
        return "Hello from Service B";
    }
}
```

### application.properties
```properties
spring.application.name=service-b
server.port=8082
eureka.client.service-url.defaultZone=http://localhost:8761/eureka
```

---

## 💡 Run Order

1. **Start Eureka Server (8761)**
2. **Start Service A (8081)**
3. **Start Service B (8082)**
4. **Start API Gateway (8080)**

## 🚀 Testing

Now you can call:

- http://localhost:8080/service-a/hello
- http://localhost:8080/service-b/hello

Gateway will automatically route requests to the respective services using Eureka Service Discovery.

## 📋 Service Ports

| Service | Port |
|---------|------|
| Eureka Server | 8761 |
| API Gateway | 8080 |
| Service A | 8081 |
| Service B | 8082 |
