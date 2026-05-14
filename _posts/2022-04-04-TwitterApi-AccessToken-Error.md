---
title: Twitter API - 개발자 계정 외 자동봇의 액세스 토큰 발급(7자리 PIN이 뜨지 않는 경우)
author: dnya0
date:   2022-04-04 15:00:00 +0900
categories: [TIL, Twitter]
tag: [Portfolio, Projects, Twitter, Api, TwitterDeveloper, Error, AccessToken]
---

### excerpt

Twitter Api Access Token Error


> 이 글은 제 네이버 블로그에서 옮겨온 글입니다.

<hr>

이런 글을 쓰지 않으려 했는데 제 이후로 트위터 api를 사용하시는 분들이 저와 같은 경우를 겪을 경우를 대비하여 적습니다.

먼저 개발자 계정이 아닌 다른 계정의 Access Token과 Access Token Secret을 발급받는 소스코드 전문입니다. 일단 자바와 파이썬을 가지고 왔습니다.

### Access Token과 Access Token Secret을 발급받는 소스코드

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

import twitter4j.Twitter;
import twitter4j.TwitterException;
import twitter4j.TwitterFactory;
import twitter4j.auth.AccessToken;
import twitter4j.auth.RequestToken;

public class HttpSourceCall {
    static AccessToken accessToken = null;
    static String accessSecret = "";
    static Twitter twitter;

    public static void main(String[] args) {
        RequestToken requestToken = null;
        AccessToken finalAccessToken = null;

        twitter = TwitterFactory.getSingleton();
        twitter.setOAuthConsumer("YOUR_API_KEY", "YOUR_API_KEY_SECRET");

        //토큰
        requestToken = null;
        try {
            requestToken = twitter.getOAuthRequestToken();
        } catch (TwitterException e) {
            e.printStackTrace();
        }

        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        System.out.println(requestToken.getAuthenticationURL());
		
        System.out.println("Enter the PIN and hit enter after you granted access");
        String pin = "";
		
        try {
            pin = br.readLine();
        } catch(IOException e1) {
            e1.printStackTrace();
        }
		
        try {
            if(pin.length() > 0) 
                accessToken = twitter.getOAuthAccessToken(requestToken, pin);
            else
                accessToken = twitter.getOAuthAccessToken(requestToken);

            String key1 = accessToken.getToken();
            String key2 = accessToken.getTokenSecret();

            System.out.println("accessToken: " + accessToken);
            System.out.println("getToken: " + key1);
            System.out.println("getTokenSecret: " + key2);
        } catch(TwitterException te) {
            if(401 == te.getStatusCode()) {
                System.out.println("unable to get the access token");
                System.out.println(te);
            }
            else te.printStackTrace();
        }
    }
}
```

```python
import tweepy
import webbrowser
import time

consumer_key = "YOUR_API_KEY"
consumer_secret = "YOUR_API_KEY_SECRET"
#개발자 계정 신청 후 받은 본인의 컨슈머 키와 컨슈머 시크릿 키 입력

callback_uri = 'oob'

auth = tweepy.OAuthHandler(consumer_key, consumer_secret, callback_uri)
redirect_url = auth.get_authorization_url()
print(redirect_url)
#앱 적용할 계정으로 전환 후 출력되는 URL 클릭

webbrowser.open(redirect_url)

user_pint_input = input("핀번호 입력 : ")   #oauth_verifier=<핀 번호> 영어 섞여잇어도 ㄱㅊ은듯?


user_pint_input
#인증 URL에 들어가서 수락 한 뒤 뜨는 PIN 번호 입력

auth.get_access_token(user_pint_input)

print("액세스 토큰 : ", auth.access_token, "\n", "액세스 시크릿 토큰 : ", auth.access_token_secret)

api = tweepy.API(auth)

me = api.me()

print(me.screen_name)
#인증 완료된 계정 아이디
```

자바와 파이썬 코드 둘 다 같은 방법으로 작동합니다.
실행시키면 링크를 얻고 (개발자 계정이 아닌) 봇계정으로 로그인 후 링크로 들어가 앱 인증을 합니다.
여기까진 괜찮은데 간혹 오류가 나거나 다른 블로그에선 알려주지 못한 상황이 발생합니다.

401 ERROR의 경우 

![디벨로퍼 설정 화면](https://user-images.githubusercontent.com/84761609/169577477-31579d4c-a3bd-482d-bc57-94475788be99.png)

트위터 디벨로퍼 내 앱의 설정을 바꿀 경우 api 키와 액세스 토큰 등을 새로 발급받아야 합니다. 

<hr>

### 7자리 PIN 번호가 뜨지 않는 경우

두 번째로 코드 실행해서 링크를 얻고, 봇 계정으로 들어가 앱 인증까지 마쳤으나 7자리 PIN 번호를 받지 못한 경우입니다.
이 경우에 대해선 인터넷에 잘 나와있지 않더라구요. 저만 PIN 번호가 안뜨는지?

블로그를 따라 잘 진행했음에도 불구하고

![코드](https://user-images.githubusercontent.com/84761609/169577923-685d2f6f-e5e9-4a2d-9c0c-348d2b5da025.png)

이 화면이 뜨지 않는 분들을 위한 설명입니다.

일단 파이썬으로 어떻게 봇 계정의 액세스 토큰을 발급받는지 설명드리겠습니다. 자바 또한 같은 방법으로 따라하시면 됩니다.


#### 1. 먼저 봇 계정으로 트위터 로그인을 한 후 위의 파이썬 코드를 실행, 링크를 받습니다.

![파이썬 코드 실행 화면](https://user-images.githubusercontent.com/84761609/169578157-a548e8eb-91ab-4a97-a22a-04adfe9ac764.png)

저는 M7로 시작하는 링크를 얻었네요.

#### 2. 해당 링크를 들어가 앱 인증을 해줍니다.

![앱 인증 화면](https://user-images.githubusercontent.com/84761609/169577505-9493044a-aa47-4954-bc44-a5542966e907.png)

앱 인증을 해주면 PIN 번호가 뜨지 않고 트위터 디벨로퍼 내에 설정해둔 callback url로 넘어갑니다. 
저는 twitter.com으로 해둬서 그냥 타임라인이 떴어요.
근데 그 위 링크를 보시면

![링크창](https://user-images.githubusercontent.com/84761609/169577549-7cc5fac8-20f0-4a5c-9f45-1a56c85b2da4.png)

주황색으로 가려둔 거 보이시나요? 저 가려진 부분을 복사해야합니다.

#### 3. oauth_verifier=의 뒷 부분을 복사해 콘솔에 핀 번호로 입력 후 토큰을 얻습니다.

![콘솔창](https://user-images.githubusercontent.com/84761609/169577518-7f570124-c2ef-438a-8131-009d1fec79ff.png)


저 부분을 복사해서 핀 번호로 입력하면 *액세스 토큰과 액세스 시크릿 토큰*이 나옵니다.

해당 토큰들이 자동봇의 액세스 및 시크릿 토큰입니다.
참고로 해당 토큰을 한 번 발급 받으면 다시 발급 안받아도 되고 계속 쓸 수 있습니다.
아무도 해당 내용에 대해 글을 써주지 않더라구요 저만 PIN 번호가 안뜨나 싶을정도로..ㅠㅠ
제 서치가 미흡한걸수도.. 5시간동안 해매며 찾은 내용입니다. 즐거운 개발되세요:)
