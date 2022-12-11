---
layout: post
title: node.js cluster ipc 통신
date: 2022-12-10
tags: node.js
---

node.js 는 이벤트 루프가 싱글스레드로 동작하기 때문에
하나의 프로세스에서 하나의 CPU 만 사용하게 된다

서버의 CPU가 여러개일 때 프로세스를 하나만 띄우면 비효율적이기 떄문에
node.js 에서는 cluster 라는 모듈을 제공해 여러개의 프로세스를 실행할 수 있게 지원하고 있다

얼마 전 API서버의 redis에서 간간히 eviction이 일어나는 이슈가 있었다
원인은 서버 배포시에 롤링배포 방식을 사용하는데
서버가 올라갈 때 redis에서 프로세스의 메모리로 캐시 데이터를 가져오는 과정에서
동시에 여러번의 조회가 일어나 발생하는 문제로 추정된다
(r5.4xlarge 인스턴스를 사용중이라서 10개의 인스턴스만 올라가도 160 번의 redis호출이 발생한다)

이에 대한 해결책 중 하나로
프로세스간 IPC 통신으로 데이터를 공유할 수 있는 방법이 있어 조사해봤다
(회사에서 node.js 12버전을 사용중이라 12.x 버전으로 확인했다)

<br/>

마스터 -> 워커 메세지 전송 예제
[https://nodejs.org/docs/latest-v12.x/api/cluster.html#cluster_worker_send_message_sendhandle_options_callback](https://nodejs.org/docs/latest-v12.x/api/cluster.html#cluster_worker_send_message_sendhandle_options_callback)

마스터 <-> 워커 양방향 메세지 전송 예제
[https://nodejs.org/docs/latest-v12.x/api/child_process.html#child_process_subprocess_send_message_sendhandle_options_callback](https://nodejs.org/docs/latest-v12.x/api/child_process.html#child_process_subprocess_send_message_sendhandle_options_callback)

<br/>

## 예제
{% highlight javascript %}
const cluster = require('cluster');
const count = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < count; i += 1) {
    const worker = cluster.fork();

    worker.on('message', function(msg) {
      console.log(msg);
    });
  }

  for (const worker of Object.values(cluster.workers)) {
    worker.send('hi worker');
  }
} else if (cluster.isWorker) {
  process.on('message', function(msg) {
    console.log(`${process.pid} worker recieve: ${msg}`);
  });

  process.send(`${process.pid} worker send: hi master`);
}
{% endhighlight %}

위 코드를 실행시켜보면 마스터와 워커들이 각각 통신하고 로그가 찍히는걸 볼 수 있다

cluster.fork() 함수를 실행하면 워커 프로세스가 실행되고
각각의 worker 객체에서 워커 프로세스와 IPC 체널을 생성한다

마스터에서 worker.send()를 하면 해당 워커에서 메세지 수신이 가능하고
워커에서 process.send()를 하면 마스터에서 메세지 수신이 가능하다

<br/>

자세한 내용은 node.js 코드에서 확인이 가능하다
(라인넘버를 첨부하면 코드가 수정되면 내용이 안맞을 수 있는데 node.js 12버전의 코드가 수정될거 같진 않아서 그냥 첨부함)

<br/>

## master 동작원리
cluster.fork 함수
[https://github.com/nodejs/node/blob/v12.x/lib/internal/cluster/master.js#L164](https://github.com/nodejs/node/blob/v12.x/lib/internal/cluster/master.js#L164)

cluster.fork 내의 createWorkerProcess() 에서 워커 프로세스를 생성
[https://github.com/nodejs/node/blob/v12.x/lib/internal/cluster/master.js#L105](https://github.com/nodejs/node/blob/v12.x/lib/internal/cluster/master.js#L105)

createWorkerProcess() 에서 return 할때 호출하는 fork 함수
[https://github.com/nodejs/node/blob/v12.x/lib/child_process.js#L67](https://github.com/nodejs/node/blob/v12.x/lib/child_process.js#L67)

fork 함수의 return 시에 spawn 호출하는 spawan 함수
[https://github.com/nodejs/node/blob/v12.x/lib/internal/child_process.js#L334](https://github.com/nodejs/node/blob/v12.x/lib/internal/child_process.js#L334)

spawan 함수의 463 라인에서 호출하는 setupChannel 함수에서 IPC 체널을 생성하고
send, onread, disconnect 함수를 정의한다
[https://github.com/nodejs/node/blob/v12.x/lib/internal/child_process.js#L531](https://github.com/nodejs/node/blob/v12.x/lib/internal/child_process.js#L531)

createWorkerProcess 에서 리턴받은 workerProcess 객체가 worker 객체의 process로 들어간다
workerProcess에 send, onread, disconnect 함수들이 정의되어 있음
[https://github.com/nodejs/node/blob/v12.x/lib/internal/cluster/master.js#L170](https://github.com/nodejs/node/blob/v12.x/lib/internal/cluster/master.js#L170)

예제처럼 worker.on으로 이벤트 리스너를 추가해놓으면
worker.send로 워커 프로세스에 메세지를 보낼 수 있음
[https://github.com/nodejs/node/blob/v12.x/lib/internal/cluster/worker.js#L44](https://github.com/nodejs/node/blob/v12.x/lib/internal/cluster/worker.js#L44)

<br/>

## worker 동작원리
node.js 프로세스가 실행될 때 가장 처음에 호출되는 함수가 어딘지 몰라서..
파악할 수 있는 범위 내에서만 분석했다

프로세스가 실행될 때 setupChildProcessIpcChannel 함수가 실행되면서 IPC 체널을 세팅한다
마스터일때는 process.env.NODE_CHANNEL_FD가 undefined 라서 체널 세팅이 되지 않는다
[https://github.com/nodejs/node/blob/v12.x/lib/internal/bootstrap/pre_execution.js#L320](https://github.com/nodejs/node/blob/v12.x/lib/internal/bootstrap/pre_execution.js#L320)

setupChildProcessIpcChannel 함수에서 호출하는 _forkChild 함수
여기서 setupChannel을 호출해 실제 IPC 체널을 세팅한다(setupChannel 함수는 마스터에서 호출한것과 같은함수 호출함)
setupChannel 의 첫번째 파라미터로 process를 넣기 때문에
process.send 함수가 정의됨
[https://github.com/nodejs/node/blob/v12.x/lib/child_process.js#L129](https://github.com/nodejs/node/blob/v12.x/lib/child_process.js#L129)

예제와같이 워커 프로세스에서 process.send 함수를 호출하면 마스터에서 메세지 수신이 가능하다
