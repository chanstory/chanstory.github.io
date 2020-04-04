---
layout: post
title: CORS 환경에서 cookie 세팅
date: 2020-04-04
tags: JAVA React
---

SpringBoot, React 환경에서 JWT로 인증을 구현중
httpOnly 플래그가 적용 된 쿠키를 사용해서 토큰값을 저장하려고 할 때
CROS문제로 쿠키 세팅이 되지 않아서 구글링 후 포스팅 함

CORS 필터를 생성 후 response header에 들어갈 값을 설정해주고 SpringSecurity 필터에 추가,
React 에서 ajax 요청을 할때 withCredentials 옵션을 설정해주면 된다.

참고 사이트 : [http://blog.naver.com/PostView.nhn?blogId=gomland&logNo=221492821285](http://blog.naver.com/PostView.nhn?blogId=gomland&logNo=221492821285)


CORS 필터
{% highlight java %}
import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletResponse;

public class CorsFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    //CORS정책 관련 설정 해줌
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) servletResponse;

        response.setHeader("Access-Control-Allow-Origin", "http://localhost:3000"); //요청이 허용될 도메인
        response.setHeader("Access-Control-Allow-Methods", "GET,POST,DELETE,PUT"); //요청이 허용될 HTTP메소드
        response.setHeader("Access-Control-Allow-Headers", "*"); //요청 시 사용할 수 있는 HTTP Header
        response.setHeader("Access-Control-Allow-Credentials", "true"); //Credential 요청 사용 유무, true로 설정할 시 Access-Control-Allow-Origin헤더에 * 를 사용 할 수 없음
        response.setHeader("Access-Control-Max-Age", "180"); //요청 결과가 캐쉬에 남아있는 시간
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {

    }
}
{% endhighlight %}

React axios 요청코드
{% highlight java %}
login(id, password) {
    return axios.get(URL, {withCredentials :  true}); //쿠키를 요청에 포함할 때 withCredentials 옵션을 true로 줘야함
}
{% endhighlight %}

현재 ie11, edge의 경우 react-create-app으로 프로젝트 생성 시 추가 설정을 해줘야 동작해서 확인 못함,
chrom에서는 도메인에 port가 붙어있을 경우 쿠키 세팅이 제한된다고 해서 확인 못함,
firefox에서 쿠키 값이 세팅 되는걸 확인 했다.

이제 react에 추가설정 해서 ie11, edge 에서 확인 후에
도메인 등록해서 chrome까지 쿠키 세팅 확인한 후에 추가기능 구현에 들어가면 될 것 같다.