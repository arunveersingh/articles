Below is the third article, focusing on null avoidance in a Spring Boot and Reactive Programming context. Links to the first and second parts are provided at the end.

---

# Spring Boot and Reactive Programming: Avoiding Null with `Mono` and `Flux`

When using Spring Boot with WebFlux and reactive programming, you work with `Mono<T>` and `Flux<T>` instead of traditional synchronous return types. These reactive types naturally support the concept of absence without resorting to `null`. Instead of returning `null`, you return an empty `Mono` or `Flux`, ensuring that downstream operators and subscribers handle the absence of data gracefully.

**If you haven’t read the previous parts yet, start here:**  
[Part 1: Eliminating `null` in Java – Core Strategies](./avoid-null-in-codebase-using-java-part-1.md)  
[Part 2: Managing Null in the Spring Framework / Spring Boot](./avoid-null-in-spring-app-part-2.md)

## Table of Contents

1. [Using `Mono` and `Flux` Instead of `null`](#using-mono-and-flux-instead-of-null)  
2. [Reactive Controller Endpoints](#reactive-controller-endpoints)  
3. [Handling Optional Values](#handling-optional-values)  
4. [Reactive Null Object Pattern](#reactive-null-object-pattern)  
5. [Validation and Reactive Endpoints](#validation-and-reactive-endpoints)  
6. [Summary](#summary)

## Using `Mono` and `Flux` Instead of `null`

In reactive programming, methods do not return `null`. Instead, you handle the absence of a value by returning `Mono.empty()` or `Flux.empty()`:

```java
import reactor.core.publisher.Mono;

public Mono<User> findUserById(String id) {
    // If user not found, return Mono.empty() instead of null
    return userRepository.findById(id).switchIfEmpty(Mono.empty());
}
```

**Advantages:**

- **Clear Absence Representation:** `Mono.empty()` explicitly signals no value.  
- **Composability:** Reactive operators handle empty signals elegantly.  
- **No Null Checks:** The reactive stream never emits `null` references.

## Reactive Controller Endpoints

In a Spring WebFlux controller, returning `Mono` or `Flux` allows for a straightforward, null-free approach:

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/users")
public class ReactiveUserController {
    private final ReactiveUserService userService;

    public ReactiveUserController(ReactiveUserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public Mono<ResponseEntity<User>> getUser(@PathVariable String id) {
        return userService.findUserById(id)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }
}
```

**Key Points:**

- **`defaultIfEmpty()`**: Provides a fallback response if the `Mono` is empty.  
- **No Null Checks:** Absence is handled by empty reactive signals, not `null`.

## Handling Optional Values

If you must deal with Java `Optional`, Project Reactor provides `Mono.justOrEmpty()`:

```java
import java.util.Optional;
import reactor.core.publisher.Mono;

public Mono<User> findUserByEmail(String email) {
    Optional<User> userOpt = userRepository.findByEmail(email);
    return Mono.justOrEmpty(userOpt); // Emits value if present, empty otherwise
}
```

**Benefit:**  
- Seamless integration with existing APIs that return `Optional`.  
- Maintains a null-free reactive pipeline.

## Reactive Null Object Pattern

The Null Object Pattern can also apply in reactive contexts. Instead of returning `null`, you emit a no-op object:

```java
public Mono<NotificationService> getNotificationService() {
    return Mono.justOrEmpty(notificationService)
               .switchIfEmpty(Mono.just(new NullNotificationService()));
}
```

**Result:**  
- All consumers receive a usable (even if no-op) `NotificationService`.  
- Eliminates the need for null checks.

## Validation and Reactive Endpoints

Bean validation and request body validation also work with reactive endpoints. If a required field is `null`, the validation fails before it reaches your handling logic:

```java
import jakarta.validation.constraints.NotNull;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Mono;

public Mono<ServerResponse> createUser(ServerRequest request) {
    return request.bodyToMono(User.class)
                  .doOnNext(user -> Objects.requireNonNull(user, "User cannot be null"))
                  .flatMap(userService::save)
                  .flatMap(savedUser -> ServerResponse.ok().bodyValue(savedUser))
                  .onErrorResume(NullPointerException.class, ex -> ServerResponse.badRequest().build());
}
```

**Note:** Typically, you’d rely on `@Validated` and `@NotNull` annotations to prevent `null` inputs from reaching this point, ensuring the pipeline remains null-free.

## Summary

In a reactive Spring Boot application, `Mono` and `Flux` inherently guide you toward a null-free architecture:

- **Use `Mono.empty()` and `Flux.empty()` to Represent Absence:** No `null` needed.  
- **Embrace Reactive Operators:** `map`, `filter`, `defaultIfEmpty()`, and others handle empty streams gracefully.  
- **Integrate Seamlessly with `Optional`:** Use `Mono.justOrEmpty()` to convert `Optional` to a non-null reactive type.  
- **Apply the Null Object Pattern:** Ensure non-blocking, predictable pipelines without `null`.  
- **Leverage Validation:** Prevent `null` at the boundaries using bean validation.

By following these principles, you naturally avoid `null` and embrace a responsive, composable, and stable reactive architecture.

---

**Continue Reading:**  
[Part 1: Eliminating `null` in Java – Core Strategies](./avoid-null-in-codebase-using-java-part-1.md)  
[Part 2: Managing Null in the Spring Framework / Spring Boot](./avoid-null-in-spring-app-part-2.md)