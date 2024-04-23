# Javascript 실행 순서

### 실행 컨텍스트와 콜 스택
> JS는 싱글 스레드 언어로, 하나의 콜 스택(call stack) 사용
> 콜스택은 실행 중인 코드의 함수 호출을 추적
> 함수가 호출되면 해당 함수의 실행 컨텍스트(execution context)가 콜 스택에 푸시된다.
> 실행이 완료되면, 해당 컨텍스트는 콜 스택에서 팝된다.


### 변수 환경, 스코프 체인
> 각 실행 컨텍스트는 변수, 스코프 체인, this 키워드에 대한 정보 포함
> 스코프 체인을 통해 현재 컨텍스트에서 변수에 접근하거나 상위 컨텍스트의 변수에 접근

### 이벤트 루프
> JS의 이벤트 루프는 콜 스택과 태스크 큐(task queue) 사이의 상호 작용 관리
> 콜 스택이 비어 있으면, 이벤트 루프는 태스크 큐에서 대기 중인 다음 이벤트를 콜 스택으로 이동


# 동기(Synchronous), 비동기(Asynchronous)

### 동기 처리
> 동기 함수는 호출된 함수의 작업이 완료될 때까지 기다리고 다음 작업 실행
> 동기적으로 실행되는 함수는 콜 스택에서 해당 함수의 실행이 끝날 때까지 다른 코드 실행 불가

### 비동기 처리
> 비동기 함수들은 즉시 완료되지 않고 callback or promise를 사용하여 나중에 결과를 처리
> 비동기 함수는 호출될 때 콜 스택에 잠시 있지만, 외부 작업 시간에 콜 스택에서 제거
> 외부 작업이 완료된 후, 해당 함수의 콜백이 태스크 큐에 추가
> 이벤트 루프는 콜 스택이 비게되면 태스크 큐의 콜백을 콜 스택으로 이동


콜 스택은 하나의 프로세서가 처리하는 쓰레드의 단위라고 본다.
해당 작업이 완료되면, 프로그램 카운터가 동작하듯이 다음 순서의 작업 큐의 내용을 준비한다.


---
# 브라우저가 HTML, CSS, JS를 처리하는 과정

### 사용자가 URL을 브라우저에 입력한 직후

URL 해석과 네트워크 요청
> URL 파싱: 사용자가 입력한 URL은 프로토콜, 호스트 이름, 포트 번호, 경로 등으로 분류
> DNS 조회: 도메인 이름 시스템(DNS)를 조회하여 호스트 이름을 IP 주소로 변환하여 웹 서버의 위치를 탐색
> HTTP 요청: 웹서버에 연결하고 HTTP 요청, 필요한 문서를 요청하는 GET 요청

서버 응답 및 문서 수신
> HTTP 응답: 서버는 요청된 자원을 찾아 HTTP 응답과 함께 보냄
> 이 응답에는 상태 코드, 콘텐츠 타입 등의 헤더 정보와 요청된 파일 내용 포함
> 문서 다운로드: HTML 파일이 브라우저로 전송됨

HTML 파싱과 DOM 트리 구축
> HTML 파싱: 브라우저는 받은 HTML 문서를 파싱하여 문서의 구조를 이해
> 파싱 과정에서 HTML 태그는 노드로 변환되고, 트리 구조인 DOM 트리 형성
> DOM 트리(Document Object Model Tree): 문서의 계층적 구조를 반영

CSS 파싱과 CSSOM 트리 구축
> CSS 파싱: 모든 CSS 파일과 style 태그 내부 CSS는 파싱되어 CSSOM 트리 구성
> CSSOM(CSS Object Model) 트리는 각 요소에 적용할 스타일 정의
> 최종적으로 렌더 트리 생성에 사용

Javascript 처리
> 스크립트 로딩: script 태그에 따라 JS 파일 로드
> defer나 async 속성이 없는 경우 JS 실행 시간 동안 HTML 파싱이 중단되기도 함
> 스크립트 실행: JS는 DOM을 조작할 수 있고, 이 과정에서 DOM 또는 CSSOM 트리가 변경됨
> 변경 사항은 렌더링에 반영

렌더 트리 구축
> 렌더 트리 생성: DOM 트리와 CSSOM 트리는 결합하여 렌더 트리 형성
> 렌더 트리는 실제로 화면에 렌더링 될 요소만을 포함
> 시각적 속성 계산: 렌더 트리의 각 노드는 시각적 위치와 크기 계산

레이아웃 처리
> 레이아웃 계산: 각 요소의 정확한 위치와 크기를 계산
> 뷰 포트 내에서 요소의 정확한 위치를 결정하게 됨

페이지 렌더링
> 페이지 그리기: 레이아웃 계산 후, 페이지의 요소들이 화면에 그려짐
> 그래픽 처리 단계를 거치며, GPU가 활용될 수 있음


# 브라우저와 운영체제의 비교

### 유사점

태스크 스캐줄링
> OS: 프로세스와 스레드를 관리하고, CPU 스케줄러를 사용하여 멀티태스킹
> 다양한 어플리케이션이 동시에 실행되는 환경을 조성
> 브라우저: 이벤트루프는 JS의 비동기 작업을 스케줄링
> 마이크로태스크와 매크로태스크를 관리하여 태스크 큐에서 작업을 콜 스택으로 이동시킴

자원관리
> OS: 메모리 관리, 디바이스 드라이버 인터페이스, 파일 시스템 관리 등 다양한 시스템 자원의 할당 및 관리 담당
> 브라우저: 메모리 관리를 통해 각 탭과 확장 프로그램이 사용하는 자원 관리, 캐시, 쿠키 등의 저장소 관리

### 차이점

실행 환경
> OS: 하드웨어 위에서 직접 실행되고, HW / SW 작업을 직접 제어하고 관리
> 브라우저: OS위에서 실행되는 응용 프로그램, WEB 기반의 컨텐츠와 App을 실행하기 위한 환경

보안 및 격리
> OS: 각 프로세스는 다른 프로세스로부터 독립적인 메모리 공간을 할당받아, 프로세스 간의 간섭을 방지하고 시스템 안정성 유지
> 브라우저: 사이트 간 스크립트 실행을 차단하는 동시에, 탭 간 격리 기술을 통해 각 탭이 서로의 데이터에 접근하지 못하게 함


---
# OS의 인터럽트와 브라우저의 매크로 태스크 큐

매크로 태스크 큐
> JS가 비동기 함수를 실행하면, 콜 스택에서 제거된다. 하지만 비동기 함수는 끝난 것이 아니므로, 매크로 태스크 큐에 삽입되어 응답이 오기를 기다린다.

마이크로 태스크 큐
> 매크로 태스크 큐가 사용하는 자료구조로 이해함
> 비동기의 chain 을 처리하기 위해 순차적인 자료구조를 추가로 사용
> 마이크로 태스크 큐가 비워지기 전에는 다음 매크로 큐의 작업이 실행되지 않는다.

### JS의 이벤트 루프에 FIFO 형태인 큐가 사용된 이유는?
> 비동기 함수의 응답 시간은 당연히 보장할 수 없다.
> 하지만 동기함수나 비동기함수일지라도 실행 순서의 보장은 대단히 중요하다.
> 따라서 브라우저는 우선순위를 파악해서 동적으로 큐의 내용을 변경하기도 한다.

> 순차적 처리 보장, 단순성과 예측 가능성, 일관성 유지를 위해 큐를 사용했다고 한다.


### 인터럽트
> HW나 SW는 CPU에 긴급하게 처리해야할 사건이 발생했음을 알릴 수 있다.
> 현재 실행중인 프로세스는 대기 상태로 전환되고, 인터럽트 서브 루틴이 실행된다.


두 작업은 모두 비동기적 사건을 처리하기 위해 '대기' 메커니즘을 사용한다.


---
