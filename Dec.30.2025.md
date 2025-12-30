1. 다대일 관계 인 이유 = 여러 개의 질문(Many)은 한 명의 사용자(One)에게 속함
과정 = 머리(Annotation): DB와 연결 설정

몸통(Fields): 저장할 데이터 항목들 (ID, 제목, 내용, 작성자)

탄생(Constructor/Builder): 새로운 질문을 생성하고 현재 시간을 기록

변화(Method): 작성된 글을 수정

2. 도메인은 단순히 데이터를 담는 통이 아니라, 규칙을 검증하는 곳이기도 합니다.

3.가장 기본적인 DB 상호작용 단계입니다.
registerBoard: 넘겨받은 질문 객체를 DB에 그대로 저장합니다.

findAllBoards: 게시판 목록을 보여주기 위해 모든 질문을 리스트로 가져옵니다.

findBoardById: 특정 글 하나만 상세히 봅니다.

4. '목록 보기' 기능을 구현할 때 사용하는 핵심 도구들입니다. 특히 데이터가 많아질 것에 대비해 페이징(Paging, 페이지 나누기)

5. 왜 List가 아니라 Page를 쓰나요?
만약 게시판에 글이 100만 개가 있는데 List<Question>으로 한꺼번에 다 가져오면 서버가 터질 수도 있습니다. Page를 쓰면 다음과 같은 데이터를 함께 돌려받을 수 있어 프론트엔드(화면) 구현이 아주 편해집니다.

content: 실제 질문 데이터 10개

totalPages: 전체 페이지가 몇 개인지 (예: "마지막 100페이지")

totalElements: 전체 게시글이 총 몇 개인지

isFirst / isLast: 지금이 첫 페이지인지, 마지막 페이지인지 여부

6. dto 작성
질문(Question) 기능을 위해 필요한 DTO는 몇 개?
가장 표준적인 구성은 4개입니다. 역할별로 덩어리를 나누어 드릴게요.

① 질문 등록용 상자 (QuestionCreateRequest)
언제 써?: 사용자가 글을 다 쓰고 "등록" 버튼을 눌렀을 때 서버로 전달되는 상자.

담는 내용: 제목, 내용 (ID나 날짜는 서버에서 알아서 하므로 필요 없음).

② 질문 수정용 상자 (QuestionUpdateRequest)
언제 써?: 글을 고치고 "수정 완료"를 눌렀을 때 서버로 전달되는 상자.

담는 내용: 수정된 제목, 수정된 내용.

③ 질문 응답용 상자 (QuestionResponse)
언제 써?: 글 목록을 보거나 상세 페이지를 클릭했을 때 사용자 화면에 뿌려줄 데이터를 담은 상자.

담는 내용: 글 번호, 제목, 내용, 작성자 이름, 조회수, 작성 날짜 등.

④ 삭제 확인용 상자 (QuestionDeleteResponse)
언제 써?: 삭제가 끝난 후 "삭제되었습니다"라는 메시지를 담아 보낼 때.

담는 내용: 성공 메시지.


7. 고치기 전 서비스
```
package com.example.qnaboard.service.quesiton;

import com.example.qnaboard.domain.question.Question;
import com.example.qnaboard.dto.question.QuestionDto;
import com.example.qnaboard.dto.question.request.QuestionCreateRequest;
import com.example.qnaboard.repository.question.QuestionRepository;
import com.example.qnaboard.repository.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class QuestionService {

    private final QuestionRepository questionRepository;
    private final UserRepository userRepository;

    public Long createQuestion(
            Long userId,
            QuestionCreateRequest request
    ) {
        
    }


    // 질문 등록
    @Transactional(readOnly = true)
    public void registerBoard(Question board) {
        questionRepository.save(board);
    }

    public Page<Question> findAllBoards(Pageable pageable) {
        return questionRepository.findAll(pageable);
    }

    public Question findBoardById(Long id) {
        return questionRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다"));
    }

    public Page<Question> findBoardByUserId(Long userId, Pageable pageable){
        return questionRepository.findByUser_Id(userId, pageable);
    }

    @Transactional
    public void updateBoard(Long id, QuestionDto requestDto) {
        Question question = findBoardById(id);
        question.update(requestDto.getTitle(), requestDto.getContent());
    }

    @Transactional
    public void deleteBoard(Long id) {
        questionRepository.deleteById(id);
    }
}



```
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED) // 무분별한 객체 생성을 막기 위해
public class Question {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    @Column(nullable = false)
    private int viewCount = 0; // 초기값을 0으로 설정

    @ManyToOne(fetch = FetchType.LAZY) // 지연 로딩 권장 (성능 최적화)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    // 생성 시간과 수정 시간을 자동으로 관리 (BaseTimeEntity가 있다면 그것을 상속받는 게 베스트입니다)
    private LocalDateTime createdAt;
    private LocalDateTime modifiedAt;

    @Builder
    public Question(String title, String content, User user) {
        this.title = title;
        this.content = content;
        this.user = user;
        this.viewCount = 0; // 빌더 사용 시 초기값 명시
        this.createdAt = LocalDateTime.now();
        this.modifiedAt = LocalDateTime.now();
    }

    public void addViewCount() {
        this.viewCount++;
    }

    public void update(String title, String content) {
        this.title = title;
        this.content = content;
        this.modifiedAt = LocalDateTime.now();
    }
}
```
```
