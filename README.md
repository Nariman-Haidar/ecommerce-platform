# ecommerce-platform

> A production-grade, microservices-based e-commerce platform built with Spring Boot, Spring Cloud, MySQL, Redis, and RabbitMQ. Designed to scale, built to learn from.

[![Build Status](https://github.com/YOUR_USERNAME/ecommerce-platform/actions/workflows/ci.yml/badge.svg)](https://github.com/YOUR_USERNAME/ecommerce-platform/actions)
[![Java](https://img.shields.io/badge/Java-21-orange?logo=java)](https://adoptium.net)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2.0-brightgreen?logo=springboot)](https://spring.io/projects/spring-boot)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue?logo=docker)](https://www.docker.com)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?logo=mysql)](https://www.mysql.com)
[![Redis](https://img.shields.io/badge/Redis-7.0-red?logo=redis)](https://redis.io)
[![RabbitMQ](https://img.shields.io/badge/RabbitMQ-3-orange?logo=rabbitmq)](https://www.rabbitmq.com)


---

## 📑 Table of Contents

- [Overview](#-overview)
- [Architecture Diagram](#-architecture-diagram)
- [Microservices](#-microservices)
- [Getting Started](#-getting-started)
- [API Documentation](#-api-documentation)
- [CI/CD Pipeline](#-cicd-pipeline)

---

## 🌟 Overview

**ShopWave** is a full-featured e-commerce backend built using the **microservices architecture pattern** — the same approach used by Amazon, Netflix, and Uber at scale.

Each service is:
- **Independent** — has its own database, its own codebase
- **Deployable separately** — update one service without touching others
- **Scalable on its own** — add more product-service instances during a product launch
- **Fault-isolated** — if review-service goes down, orders still work

This project was built as a **learning journey** from data science to backend engineering, covering all the skills modern companies require.

---

## 🏗️ Architecture Diagram

### System Overview

```
                         ┌─────────────────────────────────────────┐
                         │              CLIENT LAYER                │
                         │   Browser / Mobile App / Postman         │
                         └──────────────────┬──────────────────────┘
                                            │ HTTPS
                                            ▼
                         ┌─────────────────────────────────────────┐
                         │            API GATEWAY :8080             │
                         │   • Single Entry Point                   │
                         │   • Route requests to services           │
                         │   • JWT Token Validation                 │
                         │   • Rate Limiting                        │
                         │   • Load Balancing                       │
                         └──────────────────┬──────────────────────┘
                                            │
              ┌─────────────────────────────┼─────────────────────────────┐
              │                             │                             │
              ▼                             ▼                             ▼
   ┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
   │  user-service    │         │ product-service  │         │  order-service   │
   │     :8081        │         │     :8082        │         │     :8083        │
   │                  │         │                  │         │                  │
   │ • Register       │         │ • List products  │         │ • Place orders   │
   │ • Login/JWT      │         │ • Categories     │         │ • Order history  │
   │ • Profile        │         │ • Images         │         │ • Order status   │
   │ • Addresses      │         │ • Stock mgmt     │         │ • Order items    │
   └────────┬─────────┘         └────────┬─────────┘         └────────┬─────────┘
            │                            │                             │
            ▼                            ▼                             ▼
   ┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
   │   MySQL user_db  │         │ MySQL product_db │         │  MySQL order_db  │
   │     :3306        │         │     :3307        │         │     :3308        │
   └──────────────────┘         └──────────────────┘         └──────────────────┘

              ┌─────────────────────────────┼─────────────────────────────┐
              │                             │                             │
              ▼                             ▼                             ▼
   ┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
   │  cart-service    │         │ payment-service  │         │notification-svc  │
   │     :8084        │         │     :8085        │         │     :8086        │
   │                  │         │                  │         │                  │
   │ • Add to cart    │         │ • Process pay    │         │ • Send emails    │
   │ • Update qty     │         │ • Refunds        │         │ • SMS alerts     │
   │ • Guest carts    │         │ • Pay history    │         │ • Push notifs    │
   └────────┬─────────┘         └────────┬─────────┘         └──────────────────┘
            │                            │
            ▼                            ▼
   ┌──────────────────┐         ┌──────────────────┐
   │  Redis Cache     │         │ MySQL payment_db │
   │     :6379        │         │     :3309        │
   │ TTL: 24h         │         └──────────────────┘
   └──────────────────┘
   
              ┌─────────────────────────────┐
              │                             │
              ▼                             ▼
   ┌──────────────────┐         ┌──────────────────┐
   │  review-service  │         │ discovery-server │
   │     :8087        │         │  (Eureka) :8761  │
   │                  │         │                  │
   │ • Submit reviews │         │ • Service registry│
   │ • Helpful votes  │         │ • Health checks  │
   │ • Verified buys  │         │ • Load balancing │
   └──────────────────┘         └──────────────────┘
```

---

### Asynchronous Event Flow (RabbitMQ)

```
                        ┌─────────────────────────────────────┐
                        │           RabbitMQ :5672             │
                        │         Management UI :15672         │
                        └─────────────────────────────────────┘
                                          ▲
                                          │ publishes events
                                          │
              ┌───────────────────────────┼───────────────────────────┐
              │                           │                           │
   ORDER_CREATED event          PAYMENT_COMPLETED event    PAYMENT_FAILED event
              │                           │                           │
              ▼                           ▼                           ▼
   ┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
   │  Consumers:      │        │  Consumers:      │        │  Consumers:      │
   │ • notification   │        │ • order-service  │        │ • order-service  │
   │   (send email)   │        │   (confirm order)│        │   (cancel order) │
   │ • payment        │        │ • notification   │        │ • notification   │
   │   (init payment) │        │   (receipt email)│        │   (fail email)   │
   │ • product        │        └──────────────────┘        └──────────────────┘
   │   (reduce stock) │
   └──────────────────┘

   Additional Events:
   ─────────────────
   ORDER_SHIPPED       ──────► notification-service (shipping email + tracking)
   PRODUCT_LOW_STOCK   ──────► notification-service (admin alert email)
```

---

### Request Lifecycle — "User places an order"

```
  1. User clicks "Place Order" in the browser
                    │
                    ▼
  2. POST /api/orders  ──►  API Gateway (:8080)
                              │
                              │ validates JWT token
                              ▼
  3.                       order-service (:8083)
                              │
                    ┌─────────┼──────────┐
                    │         │          │
                    ▼         ▼          ▼
  4.          Feign call  Save order  Feign call
              product-svc  to MySQL   payment-svc
              (check stock)           (init payment)
                    │                     │
                    ▼                     ▼
  5.          Stock verified         Payment initiated
                              │
                              ▼
  6.            Publish ORDER_CREATED → RabbitMQ
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
  7.     notification    payment-svc      product-svc
         sends email     processes        deducts stock
                              │
                              ▼
  8.    Publish PAYMENT_COMPLETED → RabbitMQ
                              │
              ┌───────────────┘
              ▼
  9.    notification sends "Order Confirmed" email to user
 10.    order-service updates status to CONFIRMED

  ✅ User receives confirmation email. Order is complete.
```

---

## 🔧 Microservices

| Service | Port | Description | Database |
|---------|------|-------------|----------|
| `discovery-server` | 8761 | Eureka service registry | — |
| `api-gateway` | 8080 | Single entry point, JWT validation, routing | — |
| `user-service` | 8081 | Authentication, user profiles, addresses | MySQL `user_db` |
| `product-service` | 8082 | Product catalog, categories, inventory | MySQL `product_db` |
| `order-service` | 8083 | Order placement, order history, status tracking | MySQL `order_db` |
| `cart-service` | 8084 | Shopping cart for guests and logged-in users | MySQL `cart_db` + Redis |
| `payment-service` | 8085 | Payment processing, refunds | MySQL `payment_db` |
| `notification-service` | 8086 | Email, SMS, push notifications | MySQL `notification_db` |
| `review-service` | 8087 | Product reviews, ratings, helpful votes | MySQL `review_db` |

---

## 🚀 Getting Started

### Prerequisites

| Tool | Version | Download |
|------|---------|----------|
| JDK | 17+ | [adoptium.net](https://adoptium.net) |
| Maven | 3.8+ | Included in IntelliJ |
| Docker Desktop | Latest | [docker.com](https://www.docker.com/products/docker-desktop) |
| Git | Latest | [git-scm.com](https://git-scm.com) |
| IntelliJ IDEA | Community | [jetbrains.com](https://www.jetbrains.com/idea/download) |
| Postman | Latest | [postman.com](https://www.postman.com/downloads) |

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/ecommerce-platform.git
cd ecommerce-platform
```

### 2. Configure environment variables

```bash
cp .env.example .env
# Edit .env with your settings
```

```env
# .env.example
MYSQL_ROOT_PASSWORD=rootpassword
JWT_SECRET=your-256-bit-secret-key-here
RABBITMQ_DEFAULT_USER=admin
RABBITMQ_DEFAULT_PASS=password
REDIS_PASSWORD=redispassword
```

### 3. Start infrastructure with Docker

```bash
# Start all databases + Redis + RabbitMQ + Eureka
docker-compose up -d

# Check everything is running
docker-compose ps
```

You should see:
```
NAME               STATUS          PORTS
mysql-user         Up              0.0.0.0:3306->3306/tcp
mysql-product      Up              0.0.0.0:3307->3306/tcp
mysql-order        Up              0.0.0.0:3308->3306/tcp
mysql-cart         Up              0.0.0.0:3309->3306/tcp
mysql-payment      Up              0.0.0.0:3310->3306/tcp
redis              Up              0.0.0.0:6379->6379/tcp
rabbitmq           Up              0.0.0.0:5672->5672/tcp
eureka-server      Up              0.0.0.0:8761->8761/tcp
```

### 4. Run the services

**Option A — Run all with Docker:**
```bash
docker-compose up --build
```

**Option B — Run individually (recommended for development):**
```bash
# Terminal 1 — Start Eureka first!
cd discovery-server && mvn spring-boot:run

# Terminal 2 — API Gateway
cd api-gateway && mvn spring-boot:run

# Terminal 3+ — Start services in any order
cd user-service && mvn spring-boot:run
cd product-service && mvn spring-boot:run
cd order-service && mvn spring-boot:run
# ... etc
```


## 📖 API Documentation

### Authentication Flow

```
1. Register:  POST /api/auth/register
              Body: { email, password, firstName, lastName }
              Returns: { userId, email, message }

2. Login:     POST /api/auth/login
              Body: { email, password }
              Returns: { token, tokenType: "Bearer", expiresIn }

3. Use token: All protected endpoints need the header:
              Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

## ⚙️ CI/CD Pipeline

```
Developer pushes code to GitHub
            │
            ▼
    ┌───────────────────────────────────────────────────────┐
    │                  GitHub Actions CI                     │
    │                                                       │
    │  Trigger: push to any branch / pull request to main  │
    │                                                       │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
    │  │ Checkout │→ │  Build   │→ │   Test   │           │
    │  │  Code    │  │  Maven   │  │  JUnit5  │           │
    │  └──────────┘  └──────────┘  └──────────┘           │
    │                                    │                  │
    │                            Tests fail? → ❌ Stop     │
    │                            Tests pass? → Continue    │
    └───────────────────────────────────────────────────────┘
            │ (only on merge to main)
            ▼
    ┌───────────────────────────────────────────────────────┐
    │                  GitHub Actions CD                     │
    │                                                       │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
    │  │  Build   │→ │  Push    │→ │  Deploy  │           │
    │  │  Docker  │  │  Image   │  │  to      │           │
    │  │  Images  │  │  to GHCR │  │   │           │
    │  └──────────┘  └──────────┘  └──────────┘           │
    └───────────────────────────────────────────────────────┘

```







## 🤝 Contributing

This is a learning project. Feel free to fork it, improve it, and use it as a portfolio piece.

```bash
# Fork the repo on GitHub, then:
git clone https://github.com/YOUR_USERNAME/ecommerce-platform.git
git checkout -b feature/your-feature-name
# Make changes
git commit -m "feat: describe your change"
git push origin feature/your-feature-name
# Open a Pull Request
```

---

## 📚 Learning Resources

| Topic | Resource |
|-------|----------|
| Spring Boot | [spring.io/guides](https://spring.io/guides) |
| Microservices | [microservices.io](https://microservices.io) |
| Docker | [docs.docker.com](https://docs.docker.com) |
| JWT | [jwt.io](https://jwt.io) |
| RabbitMQ | [rabbitmq.com/tutorials](https://www.rabbitmq.com/tutorials) |
| Redis | [redis.io/docs](https://redis.io/docs) |
| GitHub Actions | [docs.github.com/actions](https://docs.github.com/en/actions) |

---



<div align="center">

**Built with ❤️ as a learning journey from Data Science to Backend Engineering**

⭐ Star this repo if it helped you learn!

</div>
