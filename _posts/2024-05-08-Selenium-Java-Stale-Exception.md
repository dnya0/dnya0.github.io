---
title: Java Selenium StaleElementReferenceException
author: dnya0
date:   2024-05-08 21:25:00 +0900
categories: [TroubleShooting, Selenium]
tag: [Study, Selenium, Crawling, Java]
---

> [에러 메시지] org.openqa.selenium.StaleElementReferenceException: stale element reference: stale element not found
{: .prompt-danger }

오늘 회사에서 크롤링을 하다가 해당 오류에 직면했다. 이 오류의 원인은 여러가지가 있기에 찾기 힘들다.

1. DOM이 변경되어 이전에 참조한 요소를 사용할 수 없는 경우
2. 해당 요소가 사라졌거나 새로 고쳐진 경우

나의 경우엔 1번이었다.

해당 오류가 난 코드는 이렇다.

```java
void search(WebDriver driver) {
    Queue<WebElement> q = new LinkedList<>();
    q.addAll(driver.findElements(By.tagName("a")));

    while(!q.isEmpty()) {
        WebElement now = q.poll();

        if (now.isEnabled() && !now.getAttribute("href").contains(url))
            continue;

        System.out.println(now.getAttribute("href"));
        driver.get(now.getAttribute("href"));

        q.addAll(driver.findElements(By.tagName("a")));
    }
}
```

<br>

driver에서 a 태그가 있는 요소만을 큐에 넣어 링크를 출력하는 코드였다. 어떻게 해결해야할까 고민이 많았으나 [이 글](https://stackoverflow.com/questions/56150033/selenium-fire-staleelementreferenceexception)을 읽고 해결할 수 있었다. 

![image](https://github.com/dnya0/dnya0.github.io/assets/84761609/3c0466ce-5dcd-4b18-91fd-96cb9028d4c9)

> When you navigate away from the first page all `WebElements` in the allLinks list get lost.

첫 번째 페이지에서 벗어나면 모든 `WebElement`가 손실된다고 한다. 이 글을 읽고 아래와 같이 수정하였다. whlie문 안에서 url을 찾는 것이 아닌, 처음부터 url을 찾아 String 목록 형태로 저장하는 것이다.

```java
void search(WebDriver driver) {
    Queue<String> q = new LinkedList<>();
    q.addAll(convertWebElementToString(driver));

    while(!q.isEmpty()) {
        String now = q.poll();

        if (now.isEnabled() && !now.contains(url))
            continue;

        if (!urlSet.isEmpty() && urlSet.contains(now))
            continue;

        System.out.println(now);
        driver.get(now);

        q.addAll(convertWebElementToString(driver));
    }
}

void convertWebElementToString(WebDriver driver) {
    return driver.findElements(By.tagName("a")).stream()
        .map(link -> link.getAttribute("href"))
        .collect(Collectors.toList());
}
```