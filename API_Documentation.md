# 📚 API Documentation

## 🛑 Essential

- **All public APIs _must_ have OpenAPI documentation.**
  Every public-facing API should be fully described using an OpenAPI specification (YAML or JSON).
  _This guarantees clarity for consumers and supports integration, testing, and maintenance._

## 👍 Good To Have

- **Document internal APIs using OpenAPI.**
  Where feasible, internal APIs should also be documented with OpenAPI specifications.
  _This improves maintainability, supports onboarding, and helps prevent knowledge silos._

## 🏆 Luxury

- **Validate implementation against API specification.**
  Use tools like [`league/openapi-psr7-validator`](https://github.com/thephpleague/openapi-psr7-validator) or similar to test your implementation against the OpenAPI spec.
  _Automated validation reduces drift between code and documentation and ensures long-term reliability._
