# 질문 작성, 수정, 삭제, 상세
## domain = 도메인(Entity)**은 이 모든 데이터와 행동이 일어나는 **'설계도의 본체'*
```
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 무분별한 객체 생성 방지
public class Question {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; // 질문 식별자 (상세보기, 수정, 삭제 시 기준)

    @Column(nullable = false)
    private String title; // 질문 제목

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content; // 질문 내용

    private String tag; // 질문 태그

    // 작성자 정보를 담기 위한 연관관계 (Member 엔티티가 있다고 가정)
    // @ManyToOne(fetch = FetchType.LAZY)
    // @JoinColumn(name = "member_id")
    // private Member author;

    private LocalDateTime createdAt; // 생성일
    private LocalDateTime modifiedAt; // 수정일

    @Builder
    public Question(String title, String content, String tag) {
        this.title = title;
        this.content = content;
        this.tag = tag;
        this.createdAt = LocalDateTime.now();
        this.modifiedAt = LocalDateTime.now();
    }

    /**
     * [수정 기능]을 위한 비즈니스 로직
     * 서비스 계층에서 setter를 쓰지 않고 이 메서드를 호출해서 수정합니다.
     */
    public void update(String title, String content, String tag) {
        this.title = title;
        this.content = content;
        this.tag = tag;
        this.modifiedAt = LocalDateTime.now();
    }
}
```

## 
