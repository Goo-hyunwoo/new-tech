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


## 도커 실행

```
version: '3.8'

services:
  redis:
    image: redis:6.2
    ports:
      - "6379:6379"
    volumes:
      - D:\fastcampus\backend\docker-compose\redis\data:/data
    command: redis-server --appendonly yes
```


### Redis CLI
> Container 내부에서 redis cli
> docker exec -it docker id redis-cli GET name "100" 
> redis-cli monitor
> redis-cli --stat (통계 정보)
> slowlog get (10ms 이상 걸리는 명령)
> info (버전 등 정보, 통계 정보 등)

> 논리적인 데이터 베이스 선택
> SELECT 0 / SELECT 1 : 0번 데이터베이스, 1번 데이터베이스 선택

> redis-benchmark -> redis 성능 테스트

# DataType
## Strings
>대표 타입으로 바이너리, 문자 데이터 저장(max: 512MB
  증가 감소에 대한 원자적 연산(increment, decrement)
  SET / SETNX / GET / MGET / INC / DEC
  O(N)으로 동작
> GET users:1:email -> "goo@test.com"
> MGET users:1:email name -> "goo@test.com" "hyunwoo"
> INCR / DECR이 필요한 이유는 Key 값을 transaction safe한 관리를 위함

#### KEY 관리
> TTL(Time To Live): 남은 시간 확인
> SET LANG java : 데이터 저장
> EXPIRE LANG 10 :  10초간 유지
> TTL LANG : 남은 시간 확인

> DEL / UNLINK : Key 삭제 (UNLINK는 비동기 삭제, LIST 등의 자료형의 삭제 시간 때문)
> MEMORY USAGE key: key의 메모리 소요량 확인

## Lists
> Linked List(String)
> LPUSH / RPUSH / LPOP / RPOP / LLEN / LRANGE
> O(1)

> Queue로 사용 시  LPOP / RPUSH
> Stack으로 사용 시 RPOP / RPUSH

> lpush books:favorites '{id:100}'   ({id:100})
> rpush books:favorites '{id:200}'   ({id:100}, {id:200})
> lrange books:favorites 0 1 -> 1) "{id:100}" 2) "{id:200}"
> lrange books:favorites 0 -1 : 전체 조회

> lpop books:favorites 1


## Sets
> Unordered collection(Unique strings)
> Unique Item

> SADD / SREM / SISMEMBER / SMEMBERS / SINTER / SCARD
> O(1) / O(N) / O(NM)
> sadd users:1:posts:100:tags java
> smembers users:1:posts:100:tags -> ("java")
> scard users:1:posts:tags -> 1


## Sorted sets
> ordered collection(unique strings)
> Leader board
> Rate limit

> ZADD / ZREM / ZRANGE (REV, BYSCORE, BYLEX and LIMIT option) 
> ZCARD / ZRANK / ZREVRANK / ZINCRBY

> zadd game:scores 100 user1
> zadd game:scores 200 user2
> zrange game:scores 0 -1 withscores => user1 100 user2 200

## Hashes
> field-value pair collections

> HSET / HGET / HMGET / HGETALL / HDEL / HINCRBY

> HSET users:1000 name goo email goo@naver.com age 35
> HGET users:1000 name -> goo
> HGETALL users:1000 -> ...
## Geospatial
> Coordinate (Lat & Lon)

> GEOADD / GEOSEARCH / GEODIST / GEOPOS
> O(logn)

> GEOADD area1 127.029 37.493 "Megabox"
> GEOADD area1 127.028 37.493 "CGV"
> GEOSEARCH area1 FROMLONLAT 127.02 37.30 BYRADIUS 5 km ASC => (empty array)
> GEOSEARCH area1 FROMLONLAT 127.02 37.30 BYRADIUS 30 km ASC => 1) "Megabox" 2) "CGV"
> GEOPOS area1 "Megabox" => 1) 127.029 2) 37.493
## Bitmap
> 0 또는 1의 값으로 이루어진 비트열
> 메모리를 적게 사용하여 대량의 데이터 저장에 유용

> SETBIT / GETBIT / BITCOUNT
> O(1) / O(N)

> SETBIT marketing:users-visit01 100 1
> SETBIT marketing:users-visit01 105 1
> GETBIT marketing:users-visit01 105 => 1
> GETBIT marketing:users-visit01 100 => 1
> GETBIT marketing:users-visit01 90 => 0

