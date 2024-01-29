# Redis

> Remote Dictionary Server

## 특징

> In-Memory Database
> 메모리에 값을 저장하는 데이터베이스로 응답 속도가 빠른 휘발성 데이터 저장소
> Key - Value 로 저장되며, 다양한 데이터 타입을 지원
> 데이터의 휘발성 때문에 백업 전략 (RDB, AOF, 사용안함)에 대한 검토 필요

> Single Thread Event Loop ( NodeJS나 웹 환경과 유사한 방식 채택 ) 
> Queue 형식으로 먼저 들어온 요청에 대해 먼저 응답함
> 그러므로 Redis에 저장하는 자료가 많아진다면, Redis가 지원하는 검색 알고리즘 검토 필요
## 사용처

> 다중 시스템에서 Session 관리
> Publisher / Subscriber 역할로 Message Queue 역할 수행
> 대용량의 DB 조회가 필요한 경우, Cache Server로서 역할 수행



