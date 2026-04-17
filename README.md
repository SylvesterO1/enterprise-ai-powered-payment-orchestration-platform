# Enterprise AI-Powered Payment Orchestration Platform

Enterprise-grade payment orchestration platform built with Java 21, Spring Boot, PostgreSQL, Kafka, Flyway, JWT/OAuth2, OpenTelemetry, Prometheus, and Grafana.

## Core capabilities

- Payment intake API
- JWT/OAuth2 secured endpoints
- Read/write scope separation
- Idempotent payment creation
- Fraud and compliance orchestration
- Outbox pattern for reliable event publishing
- Kafka-driven asynchronous processing
- DLT-based failure handling
- Flyway-managed schema evolution
- Testcontainers integration testing

## High-level architecture

```text
Client
  -> REST API / Payment Controller
  -> Payment Service
  -> PostgreSQL (payments, outbox_events)

Outbox Publisher
  -> Kafka topic: payment.received

Kafka Consumer / Orchestration
  -> validation
  -> compliance review
  -> fraud review
  -> approve / reject / fail
  -> publish payment.processed or payment.rejected

DLT Consumer
  -> handles failed consumer events
  -> marks payment as FAILED
```

## Running the service

Default startup expects these external dependencies to be available:

- PostgreSQL on `localhost:5432`
- Kafka on the configured bootstrap servers
- A JWT issuer on `http://localhost:8180/realms/payment-platform`

By default, local startup does not require a bearer token.
If you want JWT enforcement enabled, start the app with `APP_SECURITY_ENABLED=true` and send a token from the configured issuer.

For a self-contained local boot without external infrastructure, use the `local` profile:

```bash
mvn -Dtest=LocalProfileApplicationTest test
mvn -DskipTests package
java -jar target/payment-orchestration-service-1.0.0.jar --spring.profiles.active=local
```

You can also keep the default profile and disable auth temporarily:

```bash
APP_SECURITY_ENABLED=false java -jar target/payment-orchestration-service-1.0.0.jar
```

To run with auth enabled:

```bash
APP_SECURITY_ENABLED=true java -jar target/payment-orchestration-service-1.0.0.jar
```

Keycloak bootstrap:

- `docker-compose.yml` imports `infra/keycloak/payment-platform-realm.json`
- realm: `payment-platform`
- client: `payment-cli`
- test user: `payment-user`
- test password: `paymentpass`

Example token request:

```bash
curl -X POST http://localhost:8180/realms/payment-platform/protocol/openid-connect/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=password' \
  -d 'client_id=payment-cli' \
  -d 'username=payment-user' \
  -d 'password=paymentpass'
```
