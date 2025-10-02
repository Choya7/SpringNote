# Entity

## 목차
> ---
>- [Entity 정의](#Definition)
>- [Entity 역할](#the-entity's-role)
>- [관련 용어 정리](#Term)
>- [주요 어노테이션](#key-annotations)
>- [Entity 동작 흐름 예제](#example-of-entity-workflow)
>---



### Definition
> --- 
> **엔티티(Entity)** 는 애플리케이션의 도메인 모델을 데이터베이스 테이블에 매핑한 클래스입니다. <br>
> JPA에서는 @Entity로 표시된 클래스를 영속성(persistence) 대상 객체로 보고, 영속성 컨텍스트(Persistence Context)(=Hibernate의 세션)가 생명 주기를 관리합니다.
>
>---

### The Entity's role
>---
>- DB 테이블과 1:1 매핑(일반적) — 컬럼, 키, 제약 등 매핑 정보 보유
>- 상태(change)를 추적(dirty checking)하여 트랜잭션 커밋 시 자동으로 SQL 반영
>- 연관관계(조회/삽입/갱신/삭제) 관리
>- 도메인 로직(비즈니스 메서드)을 포함할 수 있음 — (엔터티가 단순 DTO가 아니라 도메인 객체 역할 수행 가능)
>---

### Term
>---
>
>---

### Key Annotations
>---
>- `@Entity` : 엔티티 클래스 표시
>- `@Table(name=...)` : 테이블 이름, 인덱스, 유니크 제약 등 설정
>- `@Id` : 기본 키
>- `@GeneratedValue(strategy = ...)` : 키 생성 전략 (IDENTITY, SEQUENCE, TABLE, AUTO)
>- `@Column` : 컬럼 속성(nullable, length 등)
>- `@Lob` : 큰 텍스트/바이너리
>- `@Enumerated` : Enum 매핑
>- `@Version` : 낙관적 락(버전 컬럼)
>- `@Transient` : 영속화 제외 필드
>- `@Embeddable` / `@Embedded` / `@EmbeddedId` : 복합값/복합키
>- 연관관계: `@OneToMany`, `@ManyToOne,` `@ManyToMany`, `@OneToOne`
>- 매핑 어노테이션: `@JoinColumn`, `@JoinTable`, `mappedBy`, `cascade`, `fetch`
>- 생명주기 콜백: `@PrePersist`, `@PostLoad`, `@PreUpdate` 등
>---

### Example of Entity Workflow
>---
>Post(1) : Comment(N) 관계
>```java
>@Entity
>@Table(name = "posts")
>public class Post {
>    @Id
>    @GeneratedValue(strategy = GenerationType.IDENTITY)
>    private Long id;
>
>    @Column(nullable=false, length=200)
>    private String title;
>
>    @Lob
>    private String content;
>
>    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
>    private List<Comment> comments = new ArrayList<>();
>
>    protected Post() {} // JPA용 기본 생성자
>
>    public Post(String title, String content) {
>        this.title = title;
>        this.content = content;
>    }
>
>    public void addComment(Comment c) {
>        comments.add(c);
>        c.setPost(this);
>    }
>
>    public void removeComment(Comment c) {
>        comments.remove(c);
>        c.setPost(null);
>    }
>
>    // getters, 비즈니스 메서드 등
>}
>```
>```java
>@Entity
>@Table(name = "comments")
>public class Comment {
>    @Id
>    @GeneratedValue(strategy = GenerationType.IDENTITY)
>    private Long id;
>
>    @Column(nullable=false)
>    private String author;
>
>    @Lob
>    private String message;
>
>    @ManyToOne(fetch = FetchType.LAZY)
>    @JoinColumn(name = "post_id", nullable=false)
>    private Post post;
>
>    protected Comment() {}
>
>    public Comment(String author, String message) {
>        this.author = author;
>        this.message = message;
>    }
>
>    void setPost(Post post) { this.post = post; }
>    // getters ...
>}
>```
>---
>`PostRepository` 예시:
>```java
>public interface PostRepository extends JpaRepository<Post, Long> {
>    List<Post> findByTitleContaining(String keyword);
>
>    @Query("select p from Post p join fetch p.comments where p.id = :id")
>    Optional<Post> findWithComments(@Param("id") Long id);
>}
>```
>`Service` 예시 (트랜잭션, DTO 권장):
>```java
>@Service
>@RequiredArgsConstructor
>public class PostService {
>    private final PostRepository postRepository;
>
>    @Transactional
>    public Post create(String title, String content) {
>        return postRepository.save(new Post(title, content));
>    }
>
>    @Transactional(readOnly = true)
>    public PostDto getWithComments(Long id) {
>        Post p = postRepository.findWithComments(id).orElseThrow(...);
>        return PostDto.fromEntity(p);
>    }
>}
>```
>### 복합키 예제 (`@Embeddable` / `@EmbeddedId`)
>```java
>@Embeddable
>public class OrderItemId implements Serializable {
>    private Long orderId;
>    private Long productId;
>    // equals, hashCode, 기본 생성자
>}
>
>@Entity
>public class OrderItem {
>    @EmbeddedId
>    private OrderItemId id;
>
>    private int quantity;
>    // ...
>}
>```
>---
