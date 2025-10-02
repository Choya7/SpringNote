# Repository

## 목차
> ---
>- [Repository 정의](#Definition)
>- [Repository 역할](#the-repository's-role)
>- [관련 용어 정리](#Term)
>- [주요 어노테이션](#key-annotations)
>- [Repository 인터페이스 구조](#repository-interface-structure)
>- [Repository 동작 흐름 예제](#example-of-repository-workflow)
>---



## Controller 개념 및 어노테이션 중심

### Definition
> ---
>- **Repository**: 데이터 접근 계층(DAO: Data Access Object)의 역할을 하는 컴포넌트.
>- **Spring Data JPA**에서는 인터페이스로 정의되며, JPA 구현체(Hibernate 등)가 자동으로 구현체를 생성해줌.
>- 데이터베이스의 **CRUD(Create, Read, Update, Delete) 연산**을 처리하는 인터페이스.
> ---

### The Repository's role
>---
>- DB와 직접 통신하는 역할 → SQL/JPA/Hibernate 호출.
>- 비즈니스 로직을 가진 Service 계층과 DB 엔티티(Entity) 사이의 중간 계층.
>- 반복적인 CRUD 코드를 자동화 (Spring Data JPA의 큰 장점).
>- 트랜잭션 관리와 결합 → Service 계층에서 @Transactional과 함께 사용.
>---

### Term
>---
>- **DAO (Data Access Object)**: Repository의 전통적인 개념. 데이터 접근 전담 객체.
>- **Entity**: DB 테이블과 매핑되는 클래스 (@Entity).
>- **Persistence Context**: JPA가 엔티티를 관리하는 1차 캐시 영역.
>- **Query Method**: 메소드 이름으로 자동 쿼리를 생성하는 기능.
>   - 예: `findByUsername(String username)` → `SELECT * FROM user WHERE username = ?`
>---

### Key Annotations
>---
>| 어노테이션                    | 설명                                                               |
>| ------------------------ | ---------------------------------------------------------------- |
>| `@Repository`            | Repository 클래스/인터페이스에 붙여 스프링 빈으로 등록. Spring의 예외 변환(AOP 기반)도 적용됨. |
>| `@EnableJpaRepositories` | JPA Repository 탐색을 활성화 (`@SpringBootApplication`에 기본 포함).        |
>| `@Transactional`         | 트랜잭션 경계를 선언 (보통 Service에 선언, 필요시 Repository에도 가능).               |
>| `@Query`                 | JPQL 또는 Native SQL 직접 작성 가능.                                     |
>| `@Modifying`             | `update`, `delete` 같은 변경 쿼리에 사용.                                 |
>---

### Repository Interface Structure
>---
>Spring Data JPA가 제공하는 기본 인터페이스 계층:
>```scss
>Repository (최상위)
> └── CrudRepository (CRUD 메소드 제공)
>      └── PagingAndSortingRepository (페이징/정렬 지원)
>           └── JpaRepository (JPA 표준 + 확장 메소드)
>```
>실무에서는 `JpaRepository<T, ID>` 를 상속하는 것이 일반적.
>
>---

### Example of Repository Workflow
>---
>#### 예시: 회원(User) 엔티티 관리
>#### (1) Entity
>```java
>import jakarta.persistence.*;
>
>@Entity
>public class User {
>    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
>    private Long id;
>
>    private String username;
>    private String email;
>}
>```
>#### (2) Repository
>```java
>import org.springframework.data.jpa.repository.JpaRepository;
>
>public interface UserRepository extends JpaRepository<User, Long> {
>    // 쿼리 메소드 예시
>    User findByUsername(String username);
>}
>```
>#### (3) Service
>```java
>import org.springframework.stereotype.Service;
>import org.springframework.transaction.annotation.Transactional;
>
>@Service
>public class UserService {
>
>    private final UserRepository userRepository;
>    public UserService(UserRepository userRepository) {
>        this.userRepository = userRepository;
>    }
>
>    @Transactional
>    public User register(String username, String email) {
>        User user = new User();
>        user.setUsername(username);
>        user.setEmail(email);
>        return userRepository.save(user); // DB 저장
>    }
>
>    public User getUser(String username) {
>        return userRepository.findByUsername(username); // DB 조회
>    }
>}
>```
>#### (4) Controller
>```java
>import org.springframework.web.bind.annotation.*;
>
>@RestController
>@RequestMapping("/users")
>public class UserController {
>    private final UserService userService;
>    public UserController(UserService userService) {
>        this.userService = userService;
>    }
>
>    @PostMapping
>    public User create(@RequestParam String username, @RequestParam String email){
>        return userService.register(username, email);
>    }
>
>    @GetMapping("/{username}")
>    public User getUser(@PathVariable String username) {
>        return userService.getUser(username);
>    }
>}
>```
>---

### 동작 순서(흐름)
>---
> 1. 클라이언트 → Controller 호출
> 2. Controller → Service 호출
> 3. Service → Repository 호출
> 4. Repository → JPA/Hibernate 통해 SQL 실행
> 5. DB 결과 반환 → Repository → Service → Controller → 클라이언트
>---