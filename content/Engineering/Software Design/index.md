---
title: "Software Design"
tags:
  - software-design
  - architecture
  - ddd
  - engineering
---

# Software Design

Kiến trúc phần mềm và design patterns cho backend systems.

## 🏗️ Architectural Patterns

### Domain-Driven Design (DDD)
- **Strategic Design**: Bounded contexts, Context mapping, Ubiquitous language
- **Tactical Design**: 
  - Entities và Value Objects
  - Aggregates và Aggregate Roots
  - Domain Events
  - Repositories
  - Domain Services vs Application Services
  - Factories

### CQRS (Command Query Responsibility Segregation)
- Command vs Query separation
- Write model vs Read model
- Event sourcing integration
- Eventual consistency patterns
- Materialized views

### Event Sourcing
- Event store patterns
- Event versioning
- Snapshots
- Projections và read models
- Event replay strategies

## 📐 Design Patterns
- [[Design Patterns/index|🎨 Design Patterns Overview]]
  - [[Design Patterns/Observer-Pattern|👁️ Observer Pattern]]

## 🎯 Best Practices
- [[SOLID|⚡ SOLID Principles]]
- Clean Architecture
- Hexagonal Architecture (Ports & Adapters)
- Microservices patterns
- API design principles (REST, gRPC)

## 📬 Message Brokers
- [[RabbitMQ|📬 RabbitMQ]]

## 📚 Resources
- Domain-Driven Design (Eric Evans)
- Implementing Domain-Driven Design (Vaughn Vernon)
- Clean Architecture (Robert C. Martin)
- Patterns of Enterprise Application Architecture (Martin Fowler)
