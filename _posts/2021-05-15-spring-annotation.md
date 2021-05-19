---
layout : post
title : "@RequestBody 는 POST 만 가능할까?"
date : 2021-05-15 17:12
subtitle : ""
categories : spring
---

## 인사말

안녕하세요. **테스트 민** 입니다. 이번 글을 시작으로 포스팅을 시작해 보려합니다.

오늘은 `@RequestBody` 는 어느 경우만 데이터 바인딩이 가능한가를 여러가지 테스트 케이스 코드를 작성하면서 탐구해 보도록 하겠습니다.

## 의문

`@RequestBody`는 *POST* 전송방식만 데이터 바인딩이 가능한가 ?

## 탐구

저는 *Spring boot* 를 사용해 *Code*를 작성하겠습니다.

우선 *Controller*입니다

```java
package com.example.demo.controller;

import lombok.AllArgsConstructor;
import lombok.ToString;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class MyController {

    @GetMapping("/insert-student")
    @ResponseBody
    public String testGet(@RequestBody Student student) {
        return student.toString();
    }
    @PostMapping("/insert-student")
    @ResponseBody
    public String testPost(@RequestBody Student student) {
        return student.toString();
    }
}

@AllArgsConstructor
@ToString
class Student {
    private String name;
    private int age;
}

```

다음은 html 을 작성해 보겠습니다.

```html
<form name="myform" method="get" action="/insert-student">
    <input type="text" name="name"/>
    <input type="text" name="age"/>
    <button type="submit">submit</button>
</form>
<button type="button" id="send"> javascript send </button>
```

다음은 html 안에 javascript 입니다

```js
$("#send").on("click", function() {
    let data = {
        name : "Hun",
        age : 20
    }
    $.ajax({
        url : "/insert-student",
        method : "get",
        data : data ,
        success : function(rtn) {
            console.log(rtn);
        }
    })
})
```

>크게는 *GET*과 *POST* 가 있지만 *POST*에도 formdata 방식과 payload 방식이 존재합니다.
해당 코드는 두가지를 모두 테스트 해보기 위해 작성하였습니다.

*Get* 방식으로 *Controller*을 호출 해보니 <span style="color:red;"> 400 ERROR</span> 발생하는것을 알 수 있습니다.

 >: Resolved [org.springframework.http.converter.HttpMessageNotReadableException: Required request body is missing: public java.lang.String com.example.demo.controller.MyController.testGet(com.example.demo.controller.Student)]

 해당 에러는 *Reqeust*의 *Body*가 필수 값이라는 의미를 담고 있습니다.

 ```html
<form name="myform" method="post" action="/insert-student">
    <input type="text" name="name"/>
    <input type="text" name="age"/>
    <button type="submit">submit</button>
</form>
<button type="button" id="send"> javascript send </button>
```

다음은 html 안에 javascript 입니다

```js
$("#send").on("click", function() {
    let data = {
        name : "Hun",
        age : 20
    }
    $.ajax({
        url : "/insert-student",
        method : "post",
        data : data ,
        success : function(rtn) {
            console.log(rtn);
        }
    })
})
```

이번엔 *POST* 방식으로 데이터를 보내 보도록 하겠습니다.

>Resolved [org.springframework.web.HttpMediaTypeNotSupportedException: Content type 'application/x-www-form-urlencoded;charset=UTF-8' not supported]

이번엔 <span style="color:red;">415 ERROR</span>가 발생하네요.
해당 *Content-Type*을 지원하지 않을 때 발생하는 에러입니다.

![415ERROR_IMAGE](/assets/images/2021-05-15/Image2.png)

이미지에서 보시면 <span style="color:red;">415 ERROR</span>가 발생한것과 *Response Header*에
*Accept* 부분이 *application/json , application/\*+json* 인것을 알 수 있습니다.
호출된 *Controller*은 *application/json*만 처리가 가능하다는 의미입니다.
(reqeust, response 를 구분해야합니다)

그렇다면 이제 *Content-Type* 을 *application/json*으로 바꾸어 보내보도록하겠습니다.

 ```html
<form name="myform" method="post" action="/insert-student" entype="application/json">
    <input type="text" name="name"/>
    <input type="text" name="age"/>
    <button type="submit">submit</button>
</form>
<button type="button" id="send"> javascript send </button>
```

다음은 html 안에 javascript 입니다

```js
$("#send").on("click", function() {
    let data = {
        name : "Hun",
        age : 20
    }
    $.ajax({
        url : "/insert-student",
        method : "post",
        data : JSON.stringify(data) ,
        contentType : "application/json",
        success : function(rtn) {
            console.log(rtn);
        }
    })
})
```

`data : JSON.stringify(data)` 으로 수정 한 이유는 javascript 객체를 *json string* 으로 바꾸기 위함입니다.

해당 코드로 *Controller* 를 호출한 결과

- form 으로 *entype*으로 *application/json*으로 변경해 보았으나 ~~해당 태그에서
application/json을 지원하지 않는 모양입니다.~~ *Content-Type*이 변경되지 않아 <span style="color:red;">ERROR</span>가 발생했습니다.

- ajax 로 *Content-Type*이 *application/json*으로 변경되어 *Controller*와 통신이 정상적으로 이루어졌습니다.

## 결론

`@RequestBody`는 *POST* 전송방식만 데이터 바인딩이 가능한가 ?

*결론은 **그렇다** 입니다*

`@RequestBody` 는 이름 그대로 *Request*의 *Body* 부분을 읽기 때문에 *Body*에 데이터를 담지 않는 *GET*방식은 사용할 수 없습니다.

`@ReqeustBody` 는 *application/json*을 지원하는 어노테이션으로 *application/json*을 지원하지 않는 *form* 태그를 사용할 수 없다. 
>(*form* 태그의 데이터를 javascript를 통해 JSON.stringify() 를 이용하여 보낼 수는 있습니다.)

따라서 `@RequestBody`는 javascript 를 통한 *Payload* 방식으로 호출을 해야 정상적으로 처리가 될 수 있다.

이상입니다.

긴 글을 읽어주셔서 감사합니다. :raised_back_of_hand:

## 참고 사항

>해당 글은 **`Spring boot`** 로 작성되었습니다.  
`Spring boot`은 설정할 때 `Gson`이라는 `Dependency`를 추가 할 수 있습니다.
해당 *Dependency*가 javascript 에서 *application/json*이라는 *Content-type* 이용하여 *Request*를 보낼 수 있게 해줍니다.
`Spring` 으로 해당 기능을 구현하기 위해서는 같은 `Gson`이나 `jackson-databind`을 pom.xml에 추가하여야 합니다. 그리고 java 에서 *Controller*를 테스트 케이스를 통하여 테스트를 하실 분들은 추가하신 `Gson` 이나 , `jackson`을
xml 또는 java config 를 통해 `MessageConvert`에 등록하셔야 <span style="color:red;">415 ERROR</span> 를 해결하 실 수 있습니다.

---

글을 쓰면서 참고한 블로그 및 사이트 (지식 공유에 깊은 감사를 드립니다. :smile: )

- [Contype과 Aceept의 차이](https://dololak.tistory.com/630)
- [GET과 POST의 차이](https://hongsii.github.io/2017/08/02/what-is-the-difference-get-and-post/)
- [JSON.stringify()를 사용하는 이유](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)
- [@RequestBody의 역활](https://mangkyu.tistory.com/72)
