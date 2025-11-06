
**Controller**

| 항목        | 설명                                                                                                                  |
| --------- | ------------------------------------------------------------------------------------------------------------------- |
| **주요 역할** | **클라이언트 요청 수신 & 응답 반환**                                                                                             |
| **위치**    | `@Controller`, `@RestController`                                                                                    |
| **담당**    | - HTTP 요청 파싱 (`@RequestBody`, `@PathVariable` 등) - 요청 검증 (`@Valid`) - Service 호출 - 응답 생성 (`ResponseEntity`, JSON 등) |
| **예시**    | `POST /users` → 사용자 생성 요청 처리                                                                                        |
| **주의점**   | **비즈니스 로직 금지!** (if, for, 계산 등 X)                                                                                   |

```java
@Controller
class UserController {

    @Autowired
    private UserService userService

    @GetMapping("/users/{id}")
    ResponseEntity getUserById(@PathVariable id) {
        userDto = userService.findUserById(id)
        if (userDto == null) {
            return ResponseEntity.notFound()
        }
        return ResponseEntity.ok(userDto)
    }

    @PostMapping("/users")
    ResponseEntity createUser(@RequestBody UserRequest request) {
        userDto = userService.createUser(request)
        return ResponseEntity.created("/users/" + userDto.id)
    }
}
```

**Service**

| 항목        | 설명                                                                                       |
| --------- | ---------------------------------------------------------------------------------------- |
| **주요 역할** | **비즈니스 로직 처리**                                                                           |
| **위치**    | `@Service`                                                                               |
| **담당**    | - 핵심 로직 (계산, 검증, 상태 변경) - 여러 Mapper 호출 조합 - 트랜잭션 관리 (`@Transactional`) - DTO ↔ Entity 변환 |
| **예시**    | 회원가입 → 이메일 중복 체크 + 비밀번호 암호화 + DB 저장                                                      |
| **주의점**   | **DB 접근 코드 금지!** (`save()`, `findById()` 등 X)                                            |
```java
@Service
class UserService {

    @Autowired
    private UserMapper userMapper

    UserDto findUserById(Long id) {
        userEntity = userMapper.selectById(id)
        if (userEntity == null) {
            return null
        }
        return convertToDto(userEntity)
    }

    UserDto createUser(UserRequest request) {
        userEntity = convertToEntity(request)
        userMapper.insert(userEntity)
        return convertToDto(userEntity)
    }

    // DTO ↔ Entity 변환 로직 (생략 가능)
    UserDto convertToDto(UserEntity entity) { ... }
    UserEntity convertToEntity(UserRequest request) { ... }
}
```

**Mapper**

| 항목        | 설명                                                            |
| --------- | ------------------------------------------------------------- |
| **주요 역할** | **DB와의 직접 소통 (CRUD)**                                         |
| **위치**    | `@Mapper` (MyBatis) 또는 `@Repository` (JPA)                    |
| **담당**    | - SQL 실행 또는 JPA 메서드 호출 - Entity ↔ DB 테이블 매핑 - 데이터 조회/저장/수정/삭제 |
| **예시**    | `userMapper.insert(user)` 또는 `userRepository.save(user)`      |
| **주의점**   | **비즈니스 로직 금지!** (검증, 계산 X)                                    |
Java
```java
@Mapper
interface UserMapper {

    UserEntity selectById(Long id)

    void insert(UserEntity user)
}
```

XML
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserMapper">

    <!-- 1. 사용자 조회 -->
    <select id="selectById" parameterType="long" 
				resultType="com.example.entity.UserEntity">
        SELECT id, name, email
        FROM users
        WHERE id = #{id}
    </select>

    <!-- 2. 사용자 삽입 (자동 생성된 ID 반환) -->
    <insert id="insert" parameterType="com.example.entity.UserEntity"
						     useGeneratedKeys="true" keyProperty="id">
        INSERT INTO users (name, email)
        VALUES (#{name}, #{email})
    </insert>
    
</mapper>
```