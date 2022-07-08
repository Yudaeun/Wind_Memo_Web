# Wind 익명 타임라인 메시지



> **익명으로 메시지를 올려 의견을 주고 받을 수 있는 웹페이지**
> 
- 메시지 창에 원하는 텍스트 메시지를 작성하고 등록하면 차례대로 메시지가 익명으로 작성됩니다.
- 작성된 메시지는 24시간이 지나고 나면 없어져서, 아무 말을 던지고 가고 싶을 때 이용할 수 있습니다.

## 🐻타임라인 페이지 개발(2021.07~2021.08)



![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc57403c-63a3-4459-891e-772b62f39d43/Untitled.png)

> 그동안 웹 개발을 어떻게 하는지, 어떤 방식으로 웹 페이지를 만들 수 있는지 몰랐기 때문에, 이제라도 웹개발을 공부해보고 싶다는 마음으로 무작정 뛰어들었습니다. 리액트네이티브와 스프링을 비슷한 시기에 공부하기 시작했는데, 어렵지만 더 재미있는 건 스프링 프레임워크였습니다. 실제로 AWS 서버에 링크를 올려 친구들에게 주면서 처음으로 내가 만든 것을 서비스하는 것에 큰 성취감을 얻을 수 있었습니다. 이 토이프로젝트를 통해 백엔드 개발자로 진로를 정하게 되었으며, 이러한 경험을 이용해 이후 학기 중에 진행했던 팀 프로젝트에서 웹개발이 필요할 때 주축을 맡아 팀을 이끌 수 있었습니다.
> 

> 🔗깃허브:  [https://github.com/Yudaeun/Wind_Memo_Web](https://github.com/Yudaeun/Wind_Memo_Web)
> 

 

## 개발환경



- Intellj
- Spring Web
- Spring Data JPA
- H2 Database
- Lombok
- MySQL Driver
- GitHub
- AWS

## 개발언어



- Java
- HTML
- CSS
- JavaScript

## 프로젝트 구조



![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/df75fdeb-5600-4655-96d8-24567e3201dd/Untitled.png)

### Web

- **Domain(Repository):** JPA를 이용해 간단히 DB에 접근하도록 관리합니다.
- **Controller:** API를 관리합니다.
- **Service:** 비즈니스 로직을 관리합니다.
- **Dto:** Request 및 Response Dto를 관리합니다.
- **View:** 웹 브라우저 상에 나타나는 화면으로, 데이터를 받아 Controller로 넘겨줍니다.

## Back-End



### 1. Repository Layer

> **Memo(Entity)에 의해 생성된 데이터베이스에 접근하는 메서드를 이용하기 위한 인터페이스입니다.**
> 

생성시간과 수정시간을 설정할 수 있는 Timestamped클래스를 만들었습니다. 아래 코드에서 createdAt 변수와 modifiedAt 변수는 각각 Time API의 `LocalDateTime` 클래스를 이용해 생성 시간과 수정시간을 만드는 것에 쓰입니다. 

```java
public class Timestamped {

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime modifiedAt;
}
```

- 그 외에, MemoRequestDto 클래스를 만들어 Request 데이터를 받을 수 있게 만들었습니다.
- `JpaRepository`를 상속받는 MemoRepository 인터페이스를 만들어 Memo Entity에 있는 데이터를 조회, 저장 및 변경, 삭제할 때 Repository 인터페이스를 이용해 Memo의 데이터를 사용할 수 있게 만들었습니다.

생성한 Timestamped를 상속받는 Memo 클래스를 만들어 username과 contents(메모 내용)이 만들어질 때, 생성 시간과 수정 시간을 자동으로 설정 되도록 만들었습니다.

### 2. Service Layer

> **실제 타임라인 메모 서비스가 이루어지는 클래스로, 실질적인 비즈니스 로직을 처리하는 계층입니다.**
> 

 

아래는 id와 RequestDto를 받아서 Memo를 Update 하는 기능을 수행하고, Id를 반환하는 MemoService 클래스입니다. id를 찾을 수 없으면 예외 메시지를 출력합니다.

```java
public class MemoService {

    private final MemoRepository memoRepository;

    @Transactional
    public Long update(Long id, MemoRequestDto requestDto) {
        Memo memo = memoRepository.findById(id).orElseThrow(
                () -> new IllegalArgumentException("아이디가 존재하지 않습니다.")
        );
        memo.update(requestDto);
        return memo.getId();
    }
}
```

### 3. Controller

> **CRUD 기능을 수행할 Controller입니다. Web Layer에 속해있습니다.**
> 

아래는 메모 생성(POST), 메모 조회(GET), 메모 변경(PUT), 메모 삭제(DELETE) API를 구현한 코드입니다.

```java
public class MemoController {

    private final MemoRepository memoRepository;
    private final MemoService memoService;

    @PostMapping("/api/memos")
    public Memo createMemo(@RequestBody MemoRequestDto requestDto) {
        Memo memo = new Memo(requestDto);
        return memoRepository.save(memo);
    }
    @GetMapping("/api/memos")
   public List<Memo> readMemo() {
        LocalDateTime start = LocalDateTime.now().minusDays(1);
        LocalDateTime end = LocalDateTime.now();
        return  memoRepository.findAllByModifiedAtBetweenOrderByModifiedAtDesc(start,end);
   }

   @PutMapping("/api/memos/{id}")
   public Long updateMemo(@PathVariable Long id,@RequestBody MemoRequestDto requestDto){
        memoService.update(id,requestDto);
        return id;
   }
   @DeleteMapping("/api/memos/{id}")
    public Long deleteMemo(@PathVariable Long id){
        memoRepository.deleteById(id);
        return id;
   }
}
```

- **createMemo():** Post Api를 구현한 메서드로, 새로운 메모 객체를 생성한 후, memoRepository에 메모를 저장합니다.
- **readMemo():** Get Api를 구현한 메서드로, 작성된 메모를 리스트로 조회할 수 있습니다. 작성된 메모는 24시간 동안만 볼 수 있게 합니다.
- **updateMemo():** Put Api를 구현한 메서드로, memoService 클래스의 update 메서드를 이용해 작성된 메모를 변경할 수 있습니다.
- **deleteMemo():** Delete Api를 구현한 메서드로, memoRepository에서 전달받은 id의 메모를 삭제합니다.

## Front-End



![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/be37676a-10bc-4866-8ff1-303841e8ea4c/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/82b06c6d-766e-435f-a5fe-58ba0a5ff7cc/Untitled.png)

### 1. 메모 생성(writePost())

- 사용자가 메모를 입력하면 내용을 확인합니다.
- 구현해둔 POST API를 이용해 새로운 메모를 생성하고 JSON data 형태를 ajax로 전송합니다.

```java
function writePost() {
            let contents = $('#contents').val();

            if (isValidContents(contents) == false) {
                return;
            }

            let username = genRandomName(10);
            let data = {'username': username, 'contents': contents};

            $.ajax({
                type: "POST",
                url: "/api/memos",
                contentType: "application/json",
                data: JSON.stringify(data),
                success: function (response) {
                    alert('메시지가 성공적으로 작성되었습니다.');
                    window.location.reload();
                }
            });
        }
```

### 2. 메모 조회(getMessages())

- 구현해둔 GET API를 이용해 작성된 메모들의 리스트를 ajax로 받아와 조회합니다.
- 각각의 메모마다 스타일을 적용합니다.

```java
function getMessages() {
            $('#cards-box').empty();
            $.ajax({
                type: "GET",
                url: "/api/memos",
                data: {},
                success: function (response) {
                    for (let i = 0; i < response.length; i++) {
                        let message = response[i];
                        let id = message['id'];
                        let username = message['username'];
                        let contents = message['contents'];
                        let modifiedAt = message['modifiedAt'];
                        addHTML(id, username, contents, modifiedAt);
                    }
                }
            });
        }
```

### 3. 메모 수정(submitEdit())

- 구현해둔 PUT API를 이용해 작성한 메모의 내용을 수정할 수 있습니다.
- 수정한 메모 내용을 확인합니다.
- 수정할 메모의 username과 contents를 확인합니다.

```java
function submitEdit(id) {
            let username = $(`#${id}-username`).text().trim();
            let contents = $(`#${id}-textarea`).val().trim();
            if (isValidContents(contents) == false) {
                return;
            }
            let data = {'username': username, 'contents': contents};

            $.ajax({
                type: "PUT",
                url: `/api/memos/${id}`,
                contentType: "application/json",
                data: JSON.stringify(data),
                success: function (response) {
                    alert('메시지 변경에 성공하였습니다.');
                    window.location.reload();
                }
            });
        }
```

### 메모 삭제(deleteOne())

- 구현해둔 DELETE API를 이용해 작성한 메모를 삭제할 수 있습니다.

```java
function deleteOne(id) {
            $.ajax({
                type: "DELETE",
                url: `/api/memos/${id}`,
                success: function (response) {
                    alert('메시지 삭제에 성공하였습니다.');
                    window.location.reload();
                }
            })
        }
```

### 그 외

- `$(document).ready(function(){})` : 페이지를 로드할 때마다 메모 리스트를 불러와서 나타냅니다.
- **addHTML():** makeMessage 함수를 이용해 각각의 메모에 스타일을 적용시켜주는 함수
- **makeMesage():** username, contents, footer(버튼) 영역으로 나누어 메모에 스타일을 적용시켜주는 함수
- **isValidContents():** 메모에 내용을 작성했는지, 140자 미만으로 작성했는지를 체크하는 함수
- **genRandomName():** userid를 A~9까지 문자 중에 랜덤으로 조합해서 생성하는 함수
- **editPost():** 수정 버튼을 클릭하면 작성 내용을 textarea에 전달하는 함수
- **showEdits():** 버튼을 클릭했을 때, contents와 edit 버튼을 숨기고, editarea와 submit, delete 버튼을 나타내는 함수
- **hideEdits():** 버튼을 클릭했을 때, editarea, submit, delete 버튼을 숨기고, contents와 edit버튼을 나타내는 함수

## AWS



![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/197a582a-247f-42cf-ad64-0d2b5bace046/Untitled.png)

- AWS RDS에서 MySQL 데이터베이스를 생성하고, 스프링 부트와 연결해서 페이지를 새로 들어가도 메모가 사라지지 않도록 만들었습니다.
- 프로젝트를 `.jar`로 빌드하고 Filezilla를 이용해 AWS EC2 서버에 업로드 해서 스프링 서버를 돌아가도록 합니다.
- 가비아에서 도메인을 구입해서 DNS를 설정하고 내 도메인 IP를 설정해주어 실제 Url을 통해 서비스를 이용할 수 있게 만들었습니다.
