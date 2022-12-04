---
layout: post
title: nginder 설치 및 사용
date: 2022-12-05
tags: Test
---

IPC를 이용한 메모리 캐싱 데이터 통신을 테스트 하던중
node.js 클러스터는 부하가 없으면 프로세스를 하나만 사용한다고 해서
서버에 부하를 발생시키기 위해 사용

부하테스트툴 중 ngrinder가 보편적으로 많이 사용하는거 같아서 써봤다

자세한 설치 및 사용법은 아래 공식문서 참조
[https://github.com/naver/ngrinder/wiki/User-Guide](https://github.com/naver/ngrinder/wiki/User-Guide)

<br/>

# 설치
버전
OS: 윈도우10
JAVA: 13.0.2
nGrinder: 3.5.7

자바가 선행적으로 설치되어 있어야 한다
나는 이미 설치되어 있어 생략

nginder war 파일을 다운로드
[https://github.com/naver/ngrinder/releases](https://github.com/naver/ngrinder/releases)

{% highlight bash %}
java -jar ngrinder-controller-3.5.7.war
{% endhighlight %}
위의 명령어로 ngrinder controller를 실행(포트지정안하면 8080 포트로 실행됨)

![img]({{ '/assets/images/start-ngrinder/ngrinder-1.JPG' | relative_url }}){: .center-image }
localhost:8080으로 접근 후 로그인
기본 id, pw는 admin이다

![img]({{ '/assets/images/start-ngrinder/ngrinder-2.JPG' | relative_url }}){: .center-image }
로그인 후에 admin - downloadAgent 클릭해서 에이전트를 다운로드

다운받은 에이전트 tar파일의 압축을 푼 후에 run_agent.bat 파일을 실행한다

![img]({{ '/assets/images/start-ngrinder/ngrinder-3.JPG' | relative_url }}){: .center-image }
admin - agentManagement 탭으로 이동

![img]({{ '/assets/images/start-ngrinder/ngrinder-4.JPG' | relative_url }}){: .center-image }
실행한 agent를 approved 해줘야함

# 사용
![img]({{ '/assets/images/start-ngrinder/ngrinder-5.JPG' | relative_url }}){: .center-image }
script 탭으로 이동 새로운 스크립트를 생성해준다

![img]({{ '/assets/images/start-ngrinder/ngrinder-6.JPG' | relative_url }}){: .center-image }
URL에 localhost:4001 입력하면 유효성이 통과가 안된다..
로컬의 서버를 테스트할때는 127.0.0.1로 입력하도록 하자

![img]({{ '/assets/images/start-ngrinder/ngrinder-7.JPG' | relative_url }}){: .center-image }
기본으로 생성된 스크립트 파일, 그루비 라는데 생긴게 자바랑 똑같다..
스크립트를 수정을 할 수도 있고 기본으로 사용해도 된다
나는 시나리오 테스트를 진행할 필요는 없어서 그냥 기본으로 사용함

스크립트 생성을 완료하려면 Save/Close 클릭

![img]({{ '/assets/images/start-ngrinder/ngrinder-8.JPG' | relative_url }}){: .center-image }
Performance Test 탭으로 이동해서 테스트를 작성한다
vuser를 입력하면 ngrinder가 적절한 Process, Threads 값을 입력한다

![img]({{ '/assets/images/start-ngrinder/ngrinder-9.JPG' | relative_url }}){: .center-image }
Save and Start 를 하면 테스트 진행가능
테스트 진행 시간도 설정할 수 있다

<br/>
설치가 어렵지 않았고 GUI로 되어있어 사용하기도 쉬웠다
헌데 groovy가 익숙하지 않아서..
스크립트를 작성해서 테스트 해야하는 경우에는
javascript로 스크립트 작성이 가능한 K6를 써보는게 어떨까 하는 생각이 든다