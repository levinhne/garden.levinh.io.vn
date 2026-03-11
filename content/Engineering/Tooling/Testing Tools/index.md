---
title: "Testing Tools"
tags:
  - testing
  - quality-assurance
  - tooling
---

# Testing Tools

## 🐍 Python Testing
- **pytest**: Testing framework
- **unittest/mock**: Built-in testing
- **coverage.py**: Code coverage
- **hypothesis**: Property-based testing
- **locust**: Load testing

## 🐹 Go Testing
- **testing**: Built-in package
- **testify**: Assertion library
- **gomock**: Mocking framework
- **go-sqlmock**: SQL mocking
- **httptest**: HTTP testing
- **gotests**: Test generation

## 🌐 API Testing
- **Postman**: API development platform
- **Insomnia**: REST client
- **k6**: Load testing tool
- **JMeter**: Performance testing

## 🧪 Testing Strategies

### Unit Testing
- Test coverage targets (>80%)
- Mock external dependencies
- Table-driven tests
- Test fixtures

### Integration Testing
- Database testing (testcontainers)
- API contract testing
- Message queue testing

### End-to-End Testing
- User flow testing
- Smoke testing
- Regression testing

## 📊 Code Quality
- **SonarQube**: Code quality platform
- **golangci-lint**: Go linters
- **pylint, flake8**: Python linters
- **mypy**: Python static type checker

## 🎯 Best Practices
- Test pyramid (nhiều unit tests, ít e2e tests)
- AAA pattern (Arrange, Act, Assert)
- FIRST principles (Fast, Independent, Repeatable, Self-validating, Timely)
- Test naming conventions
- Continuous testing trong CI/CD
