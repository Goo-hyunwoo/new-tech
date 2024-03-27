# JWT

> JSON 객체를 사용하여 가볍고 자가수용적인 방식으로 정보를 안전하게 저장할 수 있게 설계된 토큰 기반의 인증 방식(웹 표준)

> Session 대신 SPA (Single Page Application) 환경에서 많이 사용됨
> Server에 지정된 Key를 통해서만 정상적으로 암복호화가 가능
> 단, 발급된 토큰에 대해서 서버에서 임의로 사용할 수 없는 토큰으로 지정할 수 없으므로 사용자는 토큰 관리에 주의해야 함


### header
> 헤더란 JWT 타입과 암호화 알고리즘 등을 포함하며 JSON 형식으로 인코딩 됨
### payload
> 클레임(Claim) 정보를 포함하며, JSON 형식으로 인코딩 됨
> 	클레임이란 객체에 저장하고 싶은 정보로 Map 형식으로 저장됨
### signature
> header와 payload를 조합하고 비밀 키를 사용하여 생성된 서명 값
> 서명 값: 토큰의 무결성을 보장하며, JWT를 조작하지 않았다는 것을 검증

#### header와 payload는 base64로 인코딩된 정보 
#### 보안과 관련된 정보는 넣지 않는다.


### 사용 방법

#### common
```
	implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
	runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
	runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
```

#### CreateToken
```java
public LoginResponse createToken(SecurityUserVo user) {

var expiredTime = LocalDateTime.now().plusHours(accessHours);
var expiredAt = Date.from(expiredTime.atZone(ZoneId.systemDefault()).toInstant());
var key = Keys.hmacShaKeyFor(secretKey.getBytes());

var token = Jwts.builder()
	.signWith(key).expiration(expiredAt)
	.claim("id", user.getUsername())
	.claim("auth", user.getAuthorities().toString())
	.compact();

return LoginResponse.builder()
	.accessToken(token)
	.accessExpiredAt(expiredTime)
	.build();
}
```
> 토큰 만료 시간을 계산
> 키 알고리즘 선택, 256byte 이상의 키를 byte 형태로 변환하여 키 생성
> 키와 토큰 만료 시간을 입력하고, 토큰에 포함하고 싶은 정보 입력 후 토큰 생성

#### ValidateToken
```java
public boolean validateToken(String token) {
	var key = Keys.hmacShaKeyFor(secretKey.getBytes());
	var parser = Jwts.parserBuilder().setSigningKey(key).build();

	try {
		parser.parse(token);
		return true;
	} catch (Exception e) {
		if (e instanceof SignatureException) {
			throw new RuntimeException("JwtErrorCode.INVALID_TOKEN");
		} else if (e instanceof ExpiredJwtException) {
			throw new RuntimeException("JwtErrorCode.EXPIRED_TOKEN");
		} else {
			throw new RuntimeException("JwtErrorCode.UNKNOWN_TOKEN_ERROR");
		}
	}
}
```
> 암호화 때 사용한 키와 동일한 알고리즘과 키값을 이용하여 키를 생성한다.
> Parser를 키를 이용하여 생성한다.
> Token을 검증한다.
> 검증 실패 시, 에러를 통해 어떤 문제가 발생했는지 확인한다.



### 중복 로그인 방지
> JWT는 최초 인증 이후, 클라이언트 단에서 토큰을 관리하게 된다.
> 서버는 클라이언트의 요청 시, 토큰을 검증하고 정상적인 토큰 요청을 수용하여 작업을 수행한다.
> 이로 인해 클라이언트에서 토큰을 탈취당한 경우, 서버는 탈취 여부를 확인할 수 없다.

> JWT의 가장 큰 장점은 서버는 Token의 해석 발급과 검증만 담당함으로써, 저장공간을 낭비하지 않는 부분이다.
> 하지만,