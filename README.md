Event Management System Admin
이벤트 관리 시스템 (관리자)
개인정보수집 이벤트 (구-모바일배송상품)
기프트 이벤트 (구-모바일기프트)
금액지급 이벤트 (구-모바일기프트 에스원 유형 등...)

이벤트 관리 시스템 명칭에 맞도록 여러 종류의 이벤트를 진행/관리 하는 시스템 입니다. v2.0.0.0 기준으로 기프트이벤트, 금액지급이벤트, 정보수집이벤트 가 존재하며 추가적으로 확률형이벤트 가 진행 예정입니다.

유지 보수 및 추가 개발 시 앞서 기재한 대로 이벤트 진행 관리에 적합한 내용 인지 확실히 정의 후 개발 부탁드립니다.
추가로 하기에 기입 될 개발 가이드를 지켜주세요.

1. Spec
- spring boot 2.7.2
- thymeleaf
- mssql
- mybatis
- java 1.8
2. Server Information
2.1 DEV
URL: http://devems.coopnc.com:7777
IP: 58.229.234.142
DB HOST: 175.119.156.52
DB INSTANCE: EMS_DEV
DB Service account: coop_ems_test
FTP HOST: 175.119.156.51
FTP user: imgt
FTP password: imgt#@!
FTP base-dir: /ServiceCommon/ems
FTP image-path: /image
2.2 REAL
URL: https://ems.coopnc.com:7443
IP: 58.229.234.211, 58.229.234.212
DB HOST: 175.119.156.45
DB INSTANCE: EMS
DB Service account: coop_ems
FTP HOST:
175.119.156.58:5021 ( root )
58.229.234.172:21
58.229.234.173:21
58.229.234.136:21
FTP user: service_ftp
FTP password: Svc34@ftP#!
FTP base-dir: /ems
FTP image-path: /image
3. 개발 가이드
3.1 프로시저 작성 규칙
prefix 는 USP 로 고정한다.
각 프로시저의 목적에 따라 SELECT UPDATE INSERT DELETE 를 이어 작성한다.
프로시저의 서비스 목적에 따라 서비스 명칭을 이어 작성한다 EVENT MESSAGE 등...
부분적인 업데이트의 경우 해당 업데이트의 목적을 기술한다. CANCEL DELETE 등...
각 프로시저 내에서는 최대한 String (문자열) 로 조합하는 동적 쿼리는 피한다.
필수 파라미터가 누락 된 경우 THROW 99999, N'{오류내용}', 1; 을 통해 오류를 발생시킨다.
THROW 한 경우 BEGIN TRY - CATCH 구문을 통해 감싸준다.
예시
1,2,3,4)
USP_SELECT_MESSAGE_LIST
USP_SELECT_MESSAGE_LIST_COUNT
USP_UPDATE_MESSAGE_CANCEL
6)
IF
@eventSeq IS NULL THROW 99999, N'[이벤트 고유번호] 누락되었습니다.',1;
7)
BEGIN TRY
/* somthing ... */
END TRY
BEGIN CATCH
    DECLARE
@ErrorMessage NVARCHAR(4000);
    DECLARE
@ErrorSeverity INT;
    DECLARE
@ErrorState INT;
SELECT @ErrorMessage = ERROR_MESSAGE(),
       @ErrorSeverity = ERROR_SEVERITY(),
       @ErrorState = ERROR_STATE();
RAISERROR
(@ErrorMessage, @ErrorSeverity, @ErrorState)
END CATCH
3.2 자바스크립트 작성 규칙
모든 페이지는 최초 HTML 파일 호출을 기반으로 한다.
해당 페이지 내에서 최초 페이지 초기화 작업에 대한 함수를 생성/실행한다.
데이터를 조회는 모두 비동기 호출로 처리한다.
데이터 조회시 동기 처리가 필요한 부분은 async/await 방식으로 처리한다.
이벤트 핸들러 및 콜백 함수는 단행이 아닌 경우 핸들러로 분리해낸다.
스크립트는 HTML 요소 하단에 위치한다.
스크립트는 fetch, util, handler, bind, init 순으로 작성한다.
fetch: 데이터 조회 함수
util: 각 데이터를 핸들링하는 유틸성 함수
handler: 이벤트 동작을 핸들링 하는 함수
bind: 이벤트 동작을 요소에 바인딩 하는 함수
init: 초기화 함수
각 fragments 간 서로 참조하지 않는 형태로 사용한다.
fragments는 독립 적인 컴포넌트로써 개별적으로 동작 할 수 있어야 한다.
3.2.1 example
const getSomething = (params) => fetch.get("/blah/blah", params);

const dataTransform = (data) => ({data, addSomething: 1});

const handleClickButton = ({target}) => {
	// do somthing ...
}

$("button").on(click, handleClickButton);

const pageInitialize = async () => {
	const data = await getSomething({test: 1});
...
}
pageInitialize();
3.3 자바 작성 규칙
3.3.1 단일 책임 원칙을 준수 한다.
프로젝트 규모가 커짐에 따라 일부 수정이 생기는 경우 if 분기 처리로 해결함이 아니라 근본적으로 각 메소드별 메소드가 동작하는 이유와 처리 대상을 확인하고 해당 메소드의 책임이 맞는가 확인 후 작업한다.
신규 메소드/서비스 추가시에도 동일한 원칙을 준수한다.

3.3.2 Interface 활용
동일한 기능은 Interface 를 통해 추상화 하여 공통적으로 사용 할 수 있도록 처리한다.

3.3.3 각 layer 의 정의를 명확히 구분한다.
controller, service, mapper 의 구분을 명확히 하여 각 layer 에서 처리되어야 하는 변수가 다른 layer 를 침범하지 않도록 주의한다.

   ex)
   * controller 를 벗어난 `HttpSession, HttpServletRequest/Response
   * controller 내 처리/연산 로직
3.3.4 Controller 작성
restful 명명을 기본 원칙으로 url path 를 작성한다.
@RequestMapping("/getItemList") -> @GetMapping("item/list")
행위는 요청 method 를 통해 명명되고 목적을 알맞게 표기한다.
view 를 조회하는 컨트롤러의 경우 마지막 경로를 **/page 로 처리한다.
등록은 @PostMapping 수정은 @PatchMapping 으로 처리한다.
등록/수정시 Domain 객체를 @RequestBody 어노테이션을 통해 받아와 처리 한다.
3.3.5 Service 작성
List 와 Pagination 처리가 필요한 서비스의 경우 ListServiceTemplate를 extends 받아 구현하다.
service 클래스 또한 단일 책임의 원칙을 위배하지 않고 해당 서비스의 범위를 벗어나는 처리 및 연산은 진행하지 않는다.
접근제한자를 명확히 지켜 캡슐화 규칙을 준수 한다.
3.3.6 ETC
EMS 내 아이넘버3 에 대한 명칭은 I3 로 통일한다.
javadoc 이외의 메서드/함수 주석은 /** example */ 형식으로 처리한다.
기능이 추가 되면 반드시 테스트를 작성한다. (테스트를 선행으로 작성 권장)
반드시 반환 되어야 하는 값은 return type 을 Optinal<?> 로 지정한다.
4. 주요 프로세스
4.1. 인증번호 발급
4.1.1. EMS 자체발급 인증번호
   1. 인증번호 발급 요청 등록
   2. 인증번호 발급 (발급시 활성화)
4.1.2. I3 인증번호
1. 인증번호 발급 요청 등록
2. 인증번호 발급 (EMS 생성 인증번호 사용, `비활성화` 상태)
3. 사용자 활성화 `요청` (일련번호 범위로 활성화, 선택 활성화)
4. 5분 주기 배치로 1시간 이내 활성화 `요청` 상태의 인증번호 활성화
5. I3 쿠폰번호 발급
6. 활성화 `성공`
4.2. 문자 발송
4.2.1. 일반 문자 발송
1. 캠페인 등록
2. 발송 요청 정보 등록 (`발송요청` 상태)
3. `MessageComponent` 객체 형태로 엑셀 정보를 저장 한다.
4. 5분 주기 배치로 다음 5분 배치 이전까지 발송요청 목록 조회
5. 메세지 발송 이력 생성 (`응답대기` 상태)
6. `MMA` API 를 통해 발송 Agent 등록
7. 5분 주기 배치를 통해 발송 후 상태값을 `MMA` API 를 통해 조회
8. 발송 결과 업데이트
    * 발송 결과는 최대 3일까지 수신하며 3일 이내에 결과가 수신되지 않는 경우 해당 발송건은 `발송실패` 처리한다.
4.2.2. 이벤트 문자 발송
1. 이벤트 - 인증번호 메뉴 접근
2. 엑셀 양식을 통한 메세지 요청정보 등록
3. 엑셀 row 수 만큼 인증번호 생성
4. 발송 요청 일시 별 Grouping 처리
5. Grouping 처리 된 발송 요청 목록 등록
6. 각 요청 별 `MessageComponent` 데이터 등록
7. 이하 `일반 문자 발송 (4.2.1 - 4)` 프로세스와 동일
5. Git
5.1. Branch 관리
5.1.1. master
배포 브랜치 역할
현재 배포되어있는 운영서버 의 버전과 동일하게 유지
hot-fix 작업 시 해당 branch 를 base 로 branch 생성
배포 완료후 수정사항을 develop 브랜치에 merge
5.1.2 develop
개발 브랜치 역할
현재 개발서버 의 버전과 동일하게 유지
추가적인 개발요건이 들어오는경우
feature branch 생성
개발요건에 따른 개발
develop 브랜치에 push
리뷰
master 브랜치와 merge
배포
6. Package
6.1 구조
6.1.1 api
신한라이프 연동을 위한 API
6.1.2 configuration
시스템 설정에 대한 구성 클래스 interceptor, filter, exception 등
6.1.2 controller
메뉴 경로 별 controller
6.1.2 domain
됴메인 객체 (VO 보다 더 확장적인 개념)
6.1.2 mapper
was <> DB 간 호출 mapper
6.1.2 service
요청 혹은 응답 데이터의 business 로직 처리 service
6.1.2 third_party_api
I3 또는 Inicis 등의 외부 제공 API
6.1.2 utils
crypto, poi 등의 유틸성 라이브러리 또는 클래스
ETC
변동사항이 생길 경우 README.md file 을 항시 최신화로 유지한다.
EMS 관련 ERD 는 저장소 root 의 COOP_EMS_V2.0.0.exerd 파일을 참조한다.
수정사항은 별도의 commit 으로 하여 다른 수정사항들 함께 반영되지 않도록 한다.
쿼리검증 후 index, column 의 변경 사항도 ERD 에 반영한다.
퍼블리싱 요청시 src/main/resources/static/html 해당 경로에 추가되며 war 에는 포함되지 않는다.
