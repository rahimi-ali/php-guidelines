# 🏛️ Design & Architecture

## Principles

- **Modules are isolated:** Each module has clear API boundaries, just like a REST API.
- **Explicit input/output:** No leaking of internal classes/objects across module boundaries.
- **Single source for database/API access:** All through repositories.
- **Strict layering:** Each layer has clearly defined responsibilities and interactions.

## 1. **Modules and Module API Interfaces**

- **Each module exposes an API interface.**

  - Interface methods are treated like HTTP endpoints: they accept only input DTOs (or primitives) and return output DTOs (or primitives).
  - Never pass internal entities or receive classes from other modules.

- **Why:** Keeps boundaries strict, prevents accidental coupling.

**Example Structure:**

```php
// ModuleA/Api/ModuleAInterface.php
interface ModuleAInterface
{
    public function foo(FooInput $input): FooOutput;
    public function bar(BarInput $input): BarOutput;
}
```

## 2. **Repository Pattern**

- **All DB and external API access goes through repositories.**

  - No direct Eloquent/DB in services, actions, etc.

- **Why:** Makes testing easier, centralizes all data access logic.

**Example:**

```php
// UserRepository.php
interface UserRepository
{
    public function findById(int $id): User|null;

    public function save(User $user): void;

    /**
     * @param Enumerable<int, int> $ids
     * @return Enumerable<int, User>
     */
    public function getByIds(Enumerable $ids): Enumerable;
    // ...and so on
}
```

## 3. **Services**

- **Responsibilities:**

  1. Interact with the outside world (third-party APIs, etc).
  2. Implement advanced OOP patterns (Strategy, Factory, Proxy, etc).

- **Rules:**

  - Services can use other services and other module API interfaces.
  - Services _cannot_ use Actions.
  - Services _cannot_ start transactions.

- **Why:** Keeps service logic composable, reusable, and side-effect free regarding DB transactions.

**Example:**

```php
interface SmsService
{
    public function send(PhoneNumber $phone, string $message): void;
}
```

```php
interface GamificationTaskEvaluator
{
    public function evaluate(): bool;
}

class OrderCountGamificationTaskEvaluator implements GamificationTaskEvaluator
{
    public function evaluate(): bool
    {
        // ...
        return true;
    }
}
```

## 4. **Actions**

- **Every distinct user action = an Action class.**

- **Responsibilities:**

  - Contain all business logic for a single user-initiated operation.
  - Manage transactions (if needed).
  - Call services, repositories, tasks, and module APIs.

- **Rules:**

  - Only layer allowed to start/commit/rollback DB transactions.
  - Actions must not return HTTP responses.
  - Return simple structures (array, DTOs, entities, primitives).

- **Why:** Guarantees a single transaction per user action, keeps business logic atomic and clear.

**Example:**

```php
namespace Modules\User\Actions;

use Modules\User\Repositories\UserRepository;
use Modules\User\Repositories\UserRoleRepository;
use Modules\User\Tasks\GenerateAndSendOtp;

readonly class Register
{
    public function __construct(
        private UserRepository $userRepository,
        private UserRoleRepository $userRoleRepository,
        private GenerateAndSendOtp $sendOtp,
    )

    public function execute(RegistrationRequest $request): User
    {
        DB::beginTransaction();
        try {
            $existing = $this->userRepository->findByMobile($request->mobile);
            if ($$existing) {
                throw new DuplicateMobileNumberException();
            }

            $user = new User();
            $user->mobile = $request->mobile
            $user->name = $request->name;
            $this->userRepository->save($user);

            $this->userRoleRepository->addRoleToUser($user->id, Role::Customer);

            $token = $this->sendOtp->execute($request->mobile);

            DB::commit();

            return $user;
        } catch (Throwable $e) {
            DB::rollback();
            throw $e;
        }
    }
}
```

## 5. **Tasks**

- **Reusable logic called by multiple Actions.**

- Structure just like Actions (constructor DI, `execute` method), but no transactions.

- **Why:** Avoid duplication of common business steps across actions.

**Example:**

```php
namespace Modules\User\Tasks;

readonly class SendOtp
{
    public function execute(PhoneNumber $mobile): void { /* ... */ }
}
```

## 6. **Controllers**

- **Responsibilities:**

  - Validate requests (using Form Requests).
  - Call the appropriate Action.
  - Return responses using Response classes.

- **Rules:**

  - Never contain business logic.
  - Never access DB, repositories, services, etc. directly.
  - Use custom response classes (extending `JsonResponse`) for consistent error and data responses.
  - Don’t use Laravel DB validation rules; implement such checks in repositories if needed.

**Example:**

```php
readonly class AuthController
{
    public function register(RegisterRequest $request, Register $register): JsonResponse
    {
        $user = $register->execute($request);

        return new UserRegisteredResponse($user);
    }
}
```

## 7. **Error Handling**

- **Use custom exceptions extending a base exception class.**
- Map exceptions to specific response classes for consistent error handling.
- Translation keys or error codes should be handled centrally.

**Example:**

```php
class OtpResendThrottledException extends AppException
{
    public function __construct(int $remainingWaitTime)
    {
        parent::__construct(ErrorCode::OtpResendThrottled);
        $this->withClientMessage(
            Lang::get(
                'user::errors.otp_resend_throttled',
                ['retryIn' => NumberUtils::toLocaleNumber($remainingWaitTime, Lang::locale())],
            )
        )->withHttpCode(419)
    }
}
```

## 8. **DTOs (Data Transfer Objects)**

- **Define DTOs in `Modules/{Module}/Dto` namespace.**
- **Actions:**
  - Can return entities, primitives, DTOs, or arrays.
  - Can accept DTOs or primitives (never entities from outside).
- **Why:** Prevents leaking internal structures.

**Example:**

```php
readonly class RegisterUserInputDto {
    public function __construct(public string $email, public string $password) {}
}
```

## 9. **Summary Table**

| Layer      | Allowed to Call                    | Can Return           | Handles Transactions | Example Use             |
| ---------- | ---------------------------------- | -------------------- | -------------------- | ----------------------- |
| Controller | Actions                            | Response             | No                   | Glue layer, response    |
| Action     | Services, Repos, Tasks, Module API | DTO/entity/array     | Yes                  | User operations         |
| Service    | Services, Repos, Module API        | DTO/array/primitives | No                   | Third-party, OOP        |
| Task       | Repo, Services, Module API         | DTO/array/primitives | No                   | Reusable business logic |
| Repository | DB                                 | Entities/DTO/array   | No                   | DB/data access          |

# 🔥 **Why This Design?**

- **Clarity:** Every developer knows where code goes, and where to look for logic.
- **Isolation:** Modules stay clean and independent.
- **Testability:** Every layer is easy to mock or test in isolation.
- **Flexibility:** Easy to swap implementations (for testing, new features, etc).

# 🧑‍💻 **Examples**

## Example 1: Registering a User

- **Controller:** Validates request using Request class, calls `Register` Action.
- **Action:** Manages transaction, calls repository to create user, calls service to send welcome email.
- **Task:** If sending SMS/OTP code is shared, encapsulate in `GenerateAndSendOtp`.

## Example 2: Loading an Order with Products

- **OrderRepository:** Loads order and loads related order products using the `load` method on order repository.
- **Action:** Calls ProductModule API with product IDs for product details.
- **Assembles full object manually**, preserving module boundaries.

# 👩‍🏫 **Explanations & Layer "WHY"s**

- **Module API as HTTP Analogy:**
  Makes boundaries explicit. Inputs/outputs are always explicit, never leaking internals.
- **Repositories:**
  All data access is centralized—testable, swappable, no accidental tight coupling.
- **Actions:**
  The only place user-initiated workflows and transactions live. Everything is atomic.
- **Services:**
  For external side effects and advanced OOP logic; keeps business logic testable and reusable.
- **Tasks:**
  Prevent code duplication; for shared business logic.
- **Controllers:**
  Thin glue layer—just input/output handling.
- **Error Handling:**
  Consistent error responses, easy internationalization, and mapping.

# 📝 **Tips & Best Practices**

- **Keep module boundaries strict:** Verbosity is a feature, not a bug!
- **DTOs only when needed:** Use primitives for simple cases; use DTOs to clarify intent or when data gets complex.
- **Don’t “cheat” and reach across modules or layers.**
- **Use dependency injection everywhere for testability.**
