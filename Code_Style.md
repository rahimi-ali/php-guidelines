# ✅ Code Style

These rules are **not enforced by PHP-CS-Fixer**, so they must be followed manually.

## 🧱 General PHP Rules

- Use `camelCase` for all string-like internal variables and keys (unless public-facing).
- Avoid using `final` on classes unless necessary.

## 🔡 Typed Properties

- Use native property types whenever possible.
- Avoid using `@var` unless native typing is impossible (e.g., for inherited third-party classes).

## 📄 Docblocks

- Omit docblocks for fully type-hinted methods unless extra description is needed.
- When used, docblock descriptions must be **full sentences** and provide **real context**.
- Do not use **fully qualified class names** in docblocks.
- Always write `Type|null` (with `null` last), **not** `null|Type` or `?Type`.
- In method signatures, prefer `Type|null` over `?Type` (even though PHP allows both).

## 🧵 Strings

- Prefer **string interpolation**:

  ```php
  // Good
  $greeting = "Hi, I am $name.";

  // Bad
  $greeting = 'Hi, I am ' . $name . '.';
  ```

## ❓ Ternary Operators

- Break long ternary expressions across multiple lines.
- Keep them on one line only if **very short**.

## 🔍 If Statements

- Always use **curly braces**.
- Follow the **happy-path-last** pattern (fail early, return early).

## 💬 Comments

- Start single-line comments with a space after `//`.
- Use a single `*` on each line in multi-line block comments.
- Prefer expressive code over explanatory comments.

## ␣ Whitespace & Structure

- Add blank lines between logical blocks for readability.
- Don’t add extra blank lines **inside `{}`** blocks.
- Only group single-line operations **if equivalent** (e.g., migrations).

## ⚙️ Configuration

- **File names** and **keys** must use `snake_case`.
- Do **not** use `env()` outside configuration files. Create config keys for them.

## 🛠 Artisan Commands

- Use **kebab-case** for command names:

  ```bash
  php artisan delete-old-records
  ```

- Always provide **terminal feedback** after a command runs (e.g., `$this->comment('Done!');`).

## 🌐 Routing

- Public-facing URLs must use **kebab-case**.
- Route names must use **camelCase**.
- Use route methods **first** (e.g., `Route::get()->name()`).
- Route parameters should be in **camelCase**.
- Only use `/` at the beginning of a URL if it is the root (`/`).

## 📦 Controllers

- Use **singular** resource names (e.g., `PostController`).
- Stick to standard CRUD methods: `index`, `show`, `store`, `update`, `destroy`, etc.
- Extract a separate controller if additional actions are needed.

## ✅ Validation

- Always use **array syntax** for rules:

  ```php
  'email' => ['required', 'email'],
  ```

- Prefer **custom rule classes** over `Validator::extend()`.

## 🌍 Translations

- All strings (including app name, messages, etc.) must be translated.
- Use `Lang::translate()` (never hardcode output strings).

## 🏷 Naming Conventions

- **Controllers:** Singular resource + `Controller` suffix
  `UserController`, `EventDayController`
- **Invokable Controllers:** Name by action, suffixed with `Controller`
  `SendReportController`
- **Resources / Transformers:** Use plural form + `Resource`/`Transformer`
  `UsersResource`, `OrdersTransformer`
- **Jobs:** Verb-based names
  `CreateUser`, `ArchiveOldPosts`
- **Events:** Reflect timing
  `LoanApproved`, `ApprovingLoan`
- **Listeners:** Describe action, suffixed with `Listener`
  `SendInvitationMailListener`
- **Commands:** Suffixed with `Command`
  `PublishScheduledPostsCommand`

## 🛑 Errors

- Use structured error responses:

  ```json
  {
    "message": "Invalid Vendor for Promotion",
    "code": 1003
  }
  ```

- Use **module-specific error code ranges** (e.g., `1001–1999` for orders).

## 🧬 Eloquent

- Use **pagination** when returning multiple records.
- Never expose Eloquent models directly to frontend; use Transformers or Resources.
- Prefer **timestamp columns** (e.g., `published_at`) over booleans (`is_published`).
- Avoid **lazy loading**, even for `hasOne` relationships.

## 📅 Migrations

- Do **not** use `Type::cases()` or dynamic enums in migrations.
- Always hard-code enum values:

  ```php
  $table->enum('type', ['One', 'Two', 'Three']);
  ```

## 🔠 Enums

- Use **PascalCase** for enum case names, not `UPPERCASE`.

## 📚 Repositories

- `findByX()` for single item queries.
- `getByX()` or `get()` returning `Enumerable` for collections.
- `persistMany()` for batch inserts.
- `save()` for single create/update operations.
