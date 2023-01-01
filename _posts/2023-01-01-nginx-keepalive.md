---
layout: post
title: nginx TIME-WAIT 소켓 최적화
date: 2023-01-01
tags: HTTP nginx
published: true
---

Elastic Beanstalk의 EC2 인스턴스 타입을 Graviton으로 교체한 후
API서버에 많은 트래픽이 몰리면 ALB에서 5xx 에러가 발생

<br/>

## 원인파악
빈즈톡에서 로그를 확인해보니
nginx 로그에 커넥션이 실패했다는 로그가 남아있음
{% highlight bash %}
connect() to 127.0.0.1:4001 failed (99: Cannot assign requested address) while connecting to upstream
{% endhighlight %}

nginx 에러로그로 구글링을 해보면 TCP:TIME_WAIT 상태의 소켓이 문제가 된다는 것을 파악할 수 있다


현재 소켓 상태 확인
{% highlight bash %}
[ec2-user@ip-172-31-26-217 ~]$ ss -s
Total: 1201
TCP:  26809 (estab 871, closed 25923, orphaned 3, timewait 25922)
Transport Total   IP    IPv6
RAW	 0     0     0
UDP	 24    20    4
TCP	 886    874    12
INET	 910    894    16
FRAG	 0     0     0
{% endhighlight %}

현재 열려있는 TIME_WAIT 소켓들은 nginx -> 서버로 요청된 소켓들이 다수라는걸 확인할 수 있음
{% highlight bash %}
netstat -nap | grep TIME_WAIT
{% endhighlight %}

![img]({{ '/assets/images/nginx-keepalive/keepalive-1.png' | relative_url }}){: .center-image }



위 내용들을 종합해보면 장애 발생 원인을 파악할 수 있다
* Elastic Beanstalk 인스턴스를 Graviton으로 올리면서 이전에는 포함되지 않았던 nginx가 각각 인스턴스에 설치되었고
* nginx가 로컬포트를 사용해 node.js 서버로 요청을 보내면서 TIME_WAIT 상태의 소켓이 다수 생성되었고 추가적인 TCP 커넥션을 맺을 수 없게 되어 nginx, ELB에 5xx 에러가 발생

(TIME_WAIT 관련한 내용은 [CLOSE_WAIT & TIME_WAIT 최종 분석](https://tech.kakao.com/2016/04/21/closewait-timewait/#part-ii-time_wait)에 정말 자세히 나와있다)

<br/>

## 해결책
우리와 동일한 상황에서 동일한 케이스를 겪은 향로님의 해결책을 적용했다
[AWS Beanstalk을 이용한 성능 튜닝 시리즈 - 커널 파라미터 튜닝](https://jojoldu.tistory.com/319)
[AWS Beanstalk을 이용한 성능 튜닝 시리즈 - Nginx 튜닝](https://jojoldu.tistory.com/322)

* ip_local_port_range 를 수정해 사용가능한 로컬 포트의 범위를 증가
* net.ipv4.tcp_tw_reuse, net.ipv4.tcp_timestamps 설정으로 TIME_WAIT 소켓 재사용
* nginx keep-alive 적용해 커넥션을 맺은 TCP 소켓을 재사용

(nginx가 downstream일때 keepalive_timeout 기본값은 60초 인데 node.js의 keepalivetimeout 값은 5초임 upstream의 timeout이 짧을 경우 간간히 502에러가 발생할 수 있어 61초로 설정함)
참고: [Reverse Proxy, HTTP Keep-Alive Timeout, and sporadic HTTP 502s](https://iximiuz.com/en/posts/reverse-proxy-http-keep-alive-and-502s/)

<br/>

## 결과
![img]({{ '/assets/images/nginx-keepalive/keepalive-2.png' | relative_url }}){: .center-image }
![img]({{ '/assets/images/nginx-keepalive/keepalive-3.png' | relative_url }}){: .center-image }

<br/>

### 참고 블로그
[Nginx 99: Cannot assign requested address to upstream](https://gryzli.info/2018/03/05/nginx-cannot-assign-requested-address/)
[CLOSE_WAIT & TIME_WAIT 최종 분석](https://tech.kakao.com/2016/04/21/closewait-timewait/#part-ii-time_wait)
[AWS Beanstalk을 이용한 성능 튜닝 시리즈 - 커널 파라미터 튜닝](https://jojoldu.tistory.com/319)
[AWS Beanstalk을 이용한 성능 튜닝 시리즈 - Nginx 튜닝](https://jojoldu.tistory.com/322)
[nginx upstream 성능 최적화](https://brunch.co.kr/@alden/11)
[Reverse Proxy, HTTP Keep-Alive Timeout, and sporadic HTTP 502s](https://iximiuz.com/en/posts/reverse-proxy-http-keep-alive-and-502s/)
