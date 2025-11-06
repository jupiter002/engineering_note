### ResponseEntity
HTTP응답을 유연하게 제어할 수 있는 Spring래퍼.

HTTP상태코드, header, body를 한번에 설정 가능함.

**Java 표준 API와의 차이:**

| 클래스                 | 출처                                                 | 설명                                        |
| ------------------- | -------------------------------------------------- | ----------------------------------------- |
| HttpServletResponse | Java Servlet API (javax.servlet / jakarta.servlet) | Java EE 표준, 서블릿 컨테이너에서 제공                 |
| ResponseEntity      | **Spring Framework**                               | Spring의 고수준 추상화, @RestController 등과 함께 사용 |
**사용방법**

```java
ResponseEntity<T> responseEntity = new ResponseEntity<>(body, headers, status);
```
- `T`: Response Body의 타입
- `body`: Response Body 객체
- `headers`: 설정할 HTTP Header
- `status`: HTTP Response Status Code

**사용예시**

```java
// 간단한 Response 반환
@GetMapping("/api/example")
public ResponseEntity<String> getExample() {
  return new ResponseEntity<>("Data found", HttpStatus.OK);
}
```

```java
@RestController
public class UserController {
    @GetMapping("/user/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        if (user == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(user);
    }
}
```

**Method정리**

ok()

- `ok()` 메서드는 HTTP 200 Status Code와 함께 Response를 생성한다. 이는 가장 일반적으로 사용되는 메서드 중 하나이다.

```java
@GetMapping("/products")
public ResponseEntity<List<Product>> getAllProducts() {
  List<Product> products = productService.findAll();
  return ResponseEntity.ok(products); // 자동으로 status 200 반환
}
```

status()

- `status()` 메서드를 사용하면 특정 HTTP Status Code를 직접 설정할 수 있다. 이는 더 유연한 Response 처리가 필요할 때 유용하다.

```java
@PostMapping("/create")
public ResponseEntity<String> createProduct(@RequestBody Product product) {
  boolean isCreated = productService.create(product);
  return ResponseEntity.status(isCreated ? HttpStatus.CREATED : HttpStatus.BAD_REQUEST)
    .body(isCreated ? "Product created successfully" : "Error creating product");
}
```

notFound()

- `notFound()` 메서드는 HTTP 404 Status Code를 설정할 때 사용한다. 자원을 찾을 수 없을 때 클라이언트에게 명확하게 알려줄 수 있다.

```java
@GetMapping("/users/{id}")
public ResponseEntity<User> getUserById(@PathVariable String id) {
  User user = userService.findById(id);
  if (user == null) {
    return ResponseEntity.notFound().build();
  }
  return ResponseEntity.ok(user);
}
```

badRequest()

- `badRequest()` 메서드는 HTTP 400 Status Code와 함께 Response를 생성할 때 사용된다. 입력 데이터에 오류가 있을 때 유용하게 사용할 수 있다.

```java
@PostMapping("/update")
public ResponseEntity<String> updateUser(@RequestBody User user) {
  if (user.getId() == null) {
    return ResponseEntity.badRequest().body("USer ID is required");
  }
  userService.update(user);
  return ResponseEntity.ok("User updated successfully");
}
```

created()

- `created` 메서드는 HTTP 201 Status Code와 함께 Response를 생성할 때 사용된다. 새로운 resource가 성공적으로 생성되었을 때 이 메서드를 사용할 수 있다.

```java
@PostMapping("/users")
public ResponseEntity<User> addUser(@RequestBody User user) {
  User createdUser = userService.addUser(user);
  URI location = ServletUriComponentsBuilder.fromCurrentRequest()
    .path("/{id}")
    .buildAndExpand(createdUser.getId())
    .toUri();
  return ResponseEntity.created(location).body(createdUser);
}
```

noContent()

- `noContent()` 메서드는 HTTP 204 Status Code와 함께 Response를 생성할 때 사용된다. 주로 삭제 요청이나 업데이트 후 body가 필요 없는 경우에 사용된다.

```java
@DeleteMapping("/users/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable String id) {
  userService.delete(id);
  return ResponseEntity.noContent().build();
}
```

accepted()

- `accepted()` 메서드는 HTTP 202 Status Code와 함께 Response를 생성할 때 사용된다. 처리가 아직 완료되지 않았지만 시작되었음을 클라이언트에 알릴 때 사용할 수 있다.

```java
@PostMapping("/process")
public ResponseEntity<Void> processRequest(@RequestBody RequestData data) {
  processService.startProcess(data);
  return ResponseEntity.accepted().build();
}
```