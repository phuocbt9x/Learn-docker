# Day-058: System Design với Docker - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: E-commerce System

**Architecture:**
- Load Balancer: Nginx
- API Gateway: Single entry point
- Services: Product, Cart, Order, Payment
- Database: PostgreSQL (Primary + Replicas)
- Cache: Redis
- Storage: Object storage cho images

**Docker Compose:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    depends_on:
      - api-gateway
  
  api-gateway:
    image: api-gateway
    networks:
      - app-network
  
  product-service:
    image: product-service
    networks:
      - app-network
    depends_on:
      - db-primary
      - redis
  
  cart-service:
    image: cart-service
    networks:
      - app-network
    depends_on:
      - redis
  
  order-service:
    image: order-service
    networks:
      - app-network
    depends_on:
      - db-primary
  
  payment-service:
    image: payment-service
    networks:
      - app-network
  
  db-primary:
    image: postgres:14-alpine
    volumes:
      - db-data:/var/lib/postgresql/data
  
  db-replica:
    image: postgres:14-alpine
    depends_on:
      - db-primary
  
  redis:
    image: redis:7-alpine

networks:
  app-network:
    driver: bridge

volumes:
  db-data:
```

---

## ✅ BÀI TẬP 2: Social Media Platform

**Architecture:**
- Load Balancer
- API Gateway
- Services: User, Feed, Media, Messaging, Notification
- Database: PostgreSQL + MongoDB
- Cache: Redis
- Message Queue: RabbitMQ
- Storage: Object storage

**Scale estimates:**
- Users: 10M
- Requests: 50K/second
- Storage: 100TB
- Database: Sharding required

---

## ✅ BÀI TẬP 3: Analytics Platform

**Architecture:**
- Ingest: Kafka
- Processing: Stream processing
- Storage: Time-series database
- API: Query API
- Dashboards: Frontend
- Alerts: Alert service

**Data flow:**
Events → Kafka → Processing → Storage → API → Dashboards

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

