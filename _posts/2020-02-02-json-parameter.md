---
layout: post
title: json으로 넘어간 파라미터 받기
date: 2020-02-02
tags: Spring
---

회원가입 로직을 구현하던 중
React에서 axios 모듈을 사용하여 post 방식으로 파라미터를 넘기면
@RequestParam 어노테이션으로 파라미터가 받아지지 않았다.

크롬 개발자 도구로 확인해보니 

![img]({{ '/assets/images/jsonHeader.PNG' | relative_url }}){: .center-image }

Request Payload에 데이터가 담겨있는데 
Request Payload를 모르겠어서 검색해봤다..

okky 커뮤니티에서 관련글을 발견
[https://okky.kr/article/291237](https://okky.kr/article/291237)

parameter는 get 방식으로 넘어가는 쿼리스트링에 포함된 데이터를 말하는거고
Payload는 post, put 방식으로 넘어갈 때 Request의 Body에 포함되는 데이터를 말하는 것 같다.

그럼 또 의문이 생기는데 이전 회사에서 POST방식으로 요청할 때도 request.getParameter 메소드나
@RequestParam 어노테이션을 사용했던거 같아서 조사해보니

jQuery로 ajax를 보낼 때 ContentType을 명시하지 않으면 디폴트로 application/x-www-form-urlencoded; charset=UTF-8로 지정되고
이로인해 parameter로 넘어온 값을 조회할 수 있다고 한다.
참조 : [https://api.jquery.com/jQuery.ajax/](https://api.jquery.com/jQuery.ajax/)



그럼이제 Payload를 받는 법을 알아보자
스프링에서는 @RequestBody 어노테이션으로 Payload데이터를 받을 수 있다.
{% highlight scss %}
//회원가입
@RequestMapping("/join")
    public JSONObject join(@RequestBody Map payload) {
    System.out.println("payload : " + payload);
    return loginService.join(payload);
}
{% endhighlight %}

결과

![img]({{ '/assets/images/logCapture.PNG' | relative_url }}){: .center-image }
받아온 데이터가 출력되는걸 확인할 수 있다!
