# 🧪 Testing

Ensure your codebase is robust, reliable, and maintainable by following these core testing practices.

## ⚡ Test Runner

- **Framework:** Use [PHPUnit](https://phpunit.de/) for all tests.

## 📏 Coverage Guidelines

- **What to Test:**
  - **Public Interfaces:**
    - For libraries: _public methods_
    - For API backends: _API endpoints_
    - For modular monoliths: _Module Interfaces_
- **Coverage:**
  - No hard minimums, but strive for meaningful coverage to catch regressions and remove dead code.

## 🧩 Test Types

- **Feature Tests:**
  - Required for **all API endpoints** and **Module API Interface methods**.
- **Middleware:**
  - If middleware is disabled or bypassed in feature tests, ensure you write dedicated **unit tests** for it.

## 📝 Test Syntax

- **Use** the `#[Test]` attribute for all test methods.
- **Naming:**
  - Use **snake_case** for test method names to maximize readability.
