# Controller

## 목차
> ---
>#### Controller 개념 및 어노테이션 중심
>- [Controller 정의](#Definition)
>- [MVC 패턴에서 Controller 역할](#the-controllers-role-in-the-mvc-pattern)
>- [관련 용어 정리](#Term)
>- [주요 어노테이션](#key-annotations)
>- [Controller 동작 흐름 예제](#package-structure-example)
>
>#### Controller 파일 중심
>- [Controller 파일 정의 및 위치](#definition-and-location-of-controller-files)
>- [패키지 구조 예시](#package-structure-example)
>- [Controller 클래스 기본 구조](#basic-structure-of-a-controller-class)
>- [REST Controller 클래스 구조](#structure-of-a-rest-controller-class)
>- [주요 어노테이션과 사용법 요약](#summary-of-key-annotations-and-their-usage)
>- [Controller 작성 시 팁](#tips-for-writing-controller-classes)
> ---


## Controller 개념 및 어노테이션 중심

### Definition
> ---
> Spring에서 Controller는 사용자의 요청(request)을 처리하고, 서비스(Service)와 모델(Model)을 연결하여 결과를 반환하는 클래스입니다.
>
>- 요청 → Controller → Service → Model → View → 사용자
>- HTTP 요청(GET, POST 등)을 처리
>- 주로 @Controller 또는 @RestController 어노테이션 사용
> ---


### The Controller's role in the MVC pattern

> ---
>MVC 패턴에서 Controller는 중간 관리자 역할을 수행합니다.
>- Model: 데이터 및 비즈니스 로직
>- View: 사용자에게 보여지는 화면
>- Controller: 요청 처리, 서비스 호출, View/Response 반환
>흐름 예시:
>```text
>사용자 요청 → Controller → Service → Model → Controller → View/Response → 사용자
>```
> ---


### Term
| 단어             | 의미                                      |
| -------------- | --------------------------------------- |
| MVC            | Model-View-Controller 패턴. 데이터와 화면 로직 분리 |
| 요청(Request)    | 클라이언트가 서버에 보내는 HTTP 요청 |
| 응답(Response)   | 서버가 클라이언트로 보내는 결과 |
| Model          | View에 전달할 데이터 객체 |
| RedirectView   | 다른 URL로 리다이렉트  |
| ResponseEntity | HTTP 상태코드, 헤더, 바디를 포함한 응답 |

---

### Key Annotations
| 어노테이션   | 설명  |
| ----------------- | ------------------------------- |
| `@Controller`     | HTML, JSP 등 View 반환용 Controller |
| `@RestController` | JSON, XML 등 데이터 반환용 Controller |
| `@RequestMapping` | URL 매핑, GET/POST 모두 가능 |
| `@GetMapping`     | GET 요청 처리 |
| `@PostMapping`    | POST 요청 처리 |
| `@PutMapping`     | PUT 요청 처리 |
| `@DeleteMapping`  | DELETE 요청 처리 |
| `@PathVariable`   | URL 경로 변수 매핑 |
| `@RequestParam`   | 쿼리 파라미터 매핑 |
| `@RequestBody`    | 요청 JSON → 객체 매핑 |
| `@ModelAttribute` | 폼 데이터 → 객체 매핑 |
| `@Autowired`      | 의존성 주입 (서비스 등) |

>---
>---



### Package Structure Example

```java
@Controller
@RequestMapping("/home")
public class HomeController {

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("message", "안녕하세요!");
        return "index"; // index.html 또는 index.jsp
    }
}
```
---
## Controller 파일 중심

### Definition and Location of Controller Files
> ---
>- **파일 역할:** 요청 처리 및 서비스 호출  
>- **파일 확장자:** `.java`  
>- **권장 위치:** `src/main/java/[패키지]/controller/`  
>---

### Package Structure Example

>---
>```text
>src/main/java/com/example/demo/
>└── controller/
>    ├── HomeController.java
>    └── UserController.java
>```
>- HomeController.java → 기본 페이지 처리
>- UserController.java → 사용자 CRUD 처리
>---

### Basic Structure of a Controller Class
```java
package com.example.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/home")
public class HomeController {

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("message", "안녕하세요!");
        return "index";
    }
}

```


### Structure of a REST Controller Class
```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }

    @PostMapping("/")
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }

    @GetMapping("/")
    public List<User> getAllUsers() {
        return userService.findAll();
    }
}
```

### Summary of Key Annotations and Their Usage
>---
>- View 반환: @Controller + Model + return String
>- JSON 반환: @RestController + @RequestBody/@PathVariable
>- URL 매핑: @RequestMapping, @GetMapping, @PostMapping
>- 의존성 주입: @Autowired 또는 생성자 주입
>---

### Tips for Writing Controller Classes

>---
>- 비즈니스 로직은 Service에 두고, Controller는 요청/응답만 처리
>- REST API → 명사 기반 URL + HTTP 메소드 규칙
>- Controller 파일명: 명사Controller.java 형태
>- View 반환용과 REST 반환용 Controller를 분리
>---