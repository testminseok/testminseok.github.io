---
layout : post
title : "HttpURLConnection을 사용하여 받은 Stream으로 PDF 파일 보여주기"
date : 2021-05-15 17:12
subtitle : ""
categories : spring
---

## 인사말

안녕하세요. **테스트 민** 입니다.
오늘은 프로젝트를 진행하면서 배웠던 것을 기록하려고 합니다.

## 의문

현재 저의 서버에서 HttpURLConnection 을 사용하여 외부의 pdf파일을 받아와 view의 보여줘야하는 상황이 발생했습니다.
해당 기능을 어떻게 구현할지 탐구해보겠습니다.

## 탐구

저는 `Spring boot` 를 사용해 *Code*를 작성하겠습니다.

우선 *Controller*입니다

인터넷을 통해 알게되었습니다.
`Spring` 에서 제공해주는 *FileCopyUtils* 라는 것을...
해당 클래스를 통해 너무나도 쉽게 *InputStream*를 *Response*의 *OutputStream* 에 옮길 수 있었습니다.

```java
package com.example.demo.controller;

import org.springframework.http.MediaType;
import org.springframework.stereotype.Controller;
import org.springframework.util.FileCopyUtils;
import org.springframework.web.bind.annotation.GetMapping;

import javax.servlet.http.HttpServletResponse;
import java.net.HttpURLConnection;
import java.net.URL;

@Controller
public class MyController {

    @GetMapping("/report/pdf")
    public void getReportPDF(HttpServletResponse response) throws Exception{
        String callURL = ""; //Test를 원하시면 google pdf test 를 검색하시면 pdf url 이 있습니다. 
        URL url = new URL(callURL);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();

        FileCopyUtils.copy(conn.getInputStream(), response.getOutputStream());
        response.setContentType(MediaType.APPLICATION_PDF_VALUE);
        response.flushBuffer();
    }


}


```

![pdf결과창](/assets/images/2021-05-24/Image.png)

해당 코드가 정상작동 하는것을 알 수 있습니다.

## 결론

File Stream을 쉽게 관리할 수 있게 Spring 에서 Utils 클래스를 제공해준다.
해당 클래스를 사용하면 HttpURLConnection 을 통해 InputStream 을 받아와도 쉽게
View 단에 파일을 보내줄 수 있다.

Stream에 대해 더 공부를 해봐야겠습니다...

## 참고 사항

---

글을 쓰면서 참고한 블로그 및 사이트 (지식 공유에 깊은 감사를 드립니다. :smile: )

- [Spring API Document](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/FileCopyUtils.html)
