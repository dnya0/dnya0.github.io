---
title: Redis로 JWT 토큰 관리하기
author: dnya0
date:   2024-03-23 02:59:00 +0900
categories: [Backend, Spring]
tag: [Spring, Redis, JWT]
math: true
mermaid: true
image:
  path: https://github.com/dnya0/dnya0.github.io/assets/84761609/4c79b0e1-6f51-4657-ac99-c948085be91d
  alt: jwt + redis
---

## 📌 JWT 토큰 관리

JWT Token을 관리하려면 어떻게 하는 것이 좋을까? 기존에 JWT를 이용해서 로그인/회원가입 기능을 개선시킨 경험이 있다. Access Token과 Refresh Token을 발급하여 Token의 수명이 다 하면 새 Token을 발급해주면 된다. 나는 이 Token들을 관리하기 위해 Redis를 사용하였다.

### RedisConfig

```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(host, port);
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate() {
        RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setEnableTransactionSupport(true);
        return redisTemplate;
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        return new JpaTransactionManager();
    }
}

```

<br>

## 🥨 로그인 / 토큰 재발급

Refresh Token을 관리하는 Redis Templete을 구현한다.

```java
@Component
@RequiredArgsConstructor
public class RedisDao {

    private final RedisTemplate<String, String> redisTemplate;

    public void setValues(String key, String data) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set(key, data);
    }

    public void setValues(String key, String data, Duration duration) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set(key, data, duration);
    }

    public String getValues(String key) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        return values.get(key);
    }

    public void deleteValues(String key) {
        redisTemplate.delete(key);
    }

```

`redisTemplete`은 아래 코드와 같이 사용된다.

```java
@Transactional
public TokenResponse createTokensByLogin(UserResponse userResponse) throws JsonProcessingException {
    Subject atkSubject = Subject.atk(userResponse);
    Subject rtkSubject = Subject.rtk(userResponse);

    String atk = createToken(atkSubject, atkLive);
    String rtk = createToken(rtkSubject, rtkLive);

    redisDao.setValues(userResponse.getUserId(), rtk, Duration.ofMillis(rtkLive));

    return new TokenResponse(atk, rtk);
}

@Transactional
public TokenResponse reissueAtk(UserResponse userResponse) throws JsonProcessingException {
    String rtkInRedis = redisDao.getValues(userResponse.getUserId());
    if (Objects.isNull(rtkInRedis)) throw new OwnPliForbiddenException("인증 정보가 만료되었습니다.");

    Subject atkSubject = Subject.atk(userResponse);
    Subject rtkSubject = Subject.rtk(userResponse);
    String atk = createToken(atkSubject, atkLive);
    String rtk = createToken(rtkSubject, rtkLive);

    redisDao.deleteValues(userResponse.getUserId());
    redisDao.setValues(userResponse.getUserId(), rtk, Duration.ofMillis(rtkLive));
    return new TokenResponse(atk, rtk);
}
```

로그인 시 userId를 Key값으로, Refresh Token을 Value로 지정하여 Access Token이 만료되어 재발급해야 할 때 userId가 있는지, Refresh Token이 일치하는지 검증할 수 있다.

<br>

## 🍖 로그아웃

그렇다면 로그아웃 시에는 어떻게 해야할까? 나는 RedisTemplete을 하나 더 선언하여 관리해주었다. Key값으로 Access Token을 저장하고 기존에 Refresh Token을 관리하던 `redisTemplete`에서 userId를 삭제한다. 아래는 로그아웃 시 AccessToken을 저장할 `redisBlackListTemplate`을 추가한 코드이다.

```java
@Component
@RequiredArgsConstructor
public class RedisDao {

    private final RedisTemplate<String, String> redisTemplate;
    private final RedisTemplate<String, String> redisBlackListTemplate;

    public void setValues(String key, String data) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set(key, data);
    }

    public void setValues(String key, String data, Duration duration) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set(key, data, duration);
    }

    public String getValues(String key) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        return values.get(key);
    }

    public void deleteValues(String key) {
        redisTemplate.delete(key);
    }

    public void setBlackList(String key, String data) {
        ValueOperations<String, String> values = redisBlackListTemplate.opsForValue();
        values.set(key, data);
    }

    public void setBlackList(String key, String data, Duration duration) {
        ValueOperations<String, String> values = redisBlackListTemplate.opsForValue();
        values.set(key, data, duration);
    }

    public void deleteBlackList(String key) {
        redisBlackListTemplate.delete(key);
    }

    public boolean hasKeyBlackList(String key) {
        return Boolean.TRUE.equals(redisBlackListTemplate.hasKey(key));
    }

}

```

해당 코드는 아래와 같이 사용하였다.

```java
@Transactional
public void setExpirationZeroAndDeleteRtk(String accessToken, String userId) {
    redisDao.setBlackList(accessToken.replace("Bearer ", ""), "logout", Duration.ofMillis(getExpiration(accessToken)));
    redisDao.deleteValues(userId);
}

public void isLogoutUserThenThrowException(HttpServletRequest request) {
    if (redisDao.hasKeyBlackList(request.getHeader("Authorization").replace("Bearer ", ""))) {
        throw new OwnPliException("이미 로그아웃된 유저입니다.");
    }
}
```


