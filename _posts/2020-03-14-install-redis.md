---
layout: post
title: AWS 에 Redis 설치하기
date: 2020-03-14
tags: Linux Redis
---
토이프로젝트에서 JWT로 인증 기능을 구현하던중
refresh token, 로그아웃을 구현할 때 data caching이 필요해 redis를 설치하게 되었다.

참조 :[https://redis.io/topics/quickstart](https://redis.io/topics/quickstart)

아래 명령어를 입력해 redis를 다운로드 한다.
{% highlight bash %}
$ wget http://download.redis.io/redis-stable.tar.gz
$ tar xvzf redis-stable.tar.gz
$ cd redis-stable
$ make
{% endhighlight %}


설지중에 에러 사항이 발생하였는데

cc: command not found 에러 발생시
c++ 컴파일러가 없어서 발생
{% highlight bash %}
$ sudo yum -y install gcc-c++
{% endhighlight %}
명령어로 설치

zmalloc.h:51:31: error: jemalloc/jemalloc.h: No such file or directory 에러 발생시
{% highlight bash %}
$ make distclean
$ make
{% endhighlight %}
명령어 실행

설치중 No space left on device 에러도 발생
용량이 부족하거나 i노드 갯수가 부족할 때 발생하는 에러인데
{% highlight bash %}
$ df -h
{% endhighlight %}
명령어로 확인해보니 용량이 부족한 상태..

![img]({{ '/assets/images/install-redis/volume.PNG' | relative_url }}){: .center-image }
AWS에 접속해 EC2 볼륨의 용량을 늘려주고 

{% highlight bash %}
$ lsblk #파티션 확인
$ sudo growpart /dev/xvda 1  #디바이스 이름과 파티션 확인 후 입력 중간에 공백이 꼭 들어가야함
$ sudo resize2fs /dev/xvda1
{% endhighlight %}
파티션 크기와 파일 시스템을 확장 한다.
그 후 다시 make 명령어 실행

redis를 설치 해주고
{% highlight bash %}
$ sudo make install 
{% endhighlight %}
실행
{% highlight bash %}
$ redis-server
{% endhighlight %}
![img]({{ '/assets/images/install-redis/redis.PNG' | relative_url }}){: .center-image }
잘 실행된다!

![img]({{ '/assets/images/install-redis/redisTest.PNG' | relative_url }})
테스트 완료