---
layout: post
title: window에서 node.js 빌드하기
date: 2022-12-13
tags: node.js
---

cluster 모듈 코드를 분석해보기 위해
window에서 node.js 를 빌드한 내용 정리

회사에서 사용중인 버전의 코드를 확인해야 해서 12.x 버전을 빌드했음
<br/>

먼저 [https://github.com/nodejs/node](https://github.com/nodejs/node) 에서 node 레파지토리를 fork 함
이후 로컬로 fork한 레파지토리를 클론한다
<br/>

node.js 빌드 메뉴얼
[https://github.com/nodejs/node/blob/main/BUILDING.md#windows](https://github.com/nodejs/node/blob/main/BUILDING.md#windows)

나는 Manual install 로 진행했다
빌드하는법은 의외로 간단한데

* Python 3.11 이상
* Visual Studio 2019 내의 C++ 개발 워크로드 또는 Build Tools의 C++ build tools
* global PATH 로 환경변수가 설정된 Git Bash
* NASM

이 설치된 상태로 vcbuild.bat 파일을 실행하면 된다.

나는 Python, Build Tools, Git Bash 는 이미 설치되어있는 상태라 NASM 만 추가로 설치했다
<br/>
빌드중에 NASM를 찾지 못한다는 에러가 발생
{% highlight bash %}
Could not find NASM, install it or build with openssl-no-asm. See BUILDING.md.
{% endhighlight %}
<br/>
vcbuild.bat을 확인해보니 find_nasm.cmd 파일에서 NASM 를 찾고있는거 같다
{% highlight bash %}
REM NASM is only needed on IA32 and x86_64.
if not defined openssl_no_asm if "%target_arch%" NEQ "arm64" call tools\msvs\find_nasm.cmd
if errorlevel 1 echo Could not find NASM, install it or build with openssl-no-asm. See BUILDING.md.
{% endhighlight %}
<br/>
find_nasm.cmd 을 확인해보면
%ProgramFiles%\NASM\nasm.exe 또는 %ProgramFiles(x86)%\NASM\nasm.exe 의 경로에서 NASM를 찾고있음
{% highlight bash %}
@IF NOT DEFINED DEBUG_HELPER @ECHO OFF

ECHO Looking for NASM

FOR /F "delims=" %%a IN ('where nasm 2^> NUL') DO (
  EXIT /B 0
)

IF EXIST "%ProgramFiles%\NASM\nasm.exe" (
  SET "Path=%Path%;%ProgramFiles%\NASM"
  EXIT /B 0
)

IF EXIST "%ProgramFiles(x86)%\NASM\nasm.exe" (
  SET "Path=%Path%;%ProgramFiles(x86)%\NASM"
  EXIT /B 0
)

EXIT /B 1
{% endhighlight %}

NASM 설치시에 아무런 생각없이 이상한 경로에 설치해서 파일을 못찾았다..
다시 설치파일을 관리자권한으로 실행해서 %ProgramFiles%\NASM 경로에 설치(64비트 파일을 설치했음
<br/>
다시 vcbuild.bat 파일을 실행
이번에는 v10.0, v8.1 의 SDK를 못찾는다는 에러 발생
{% highlight bash %}
Visual Studio 2017 requires any SDK of ['v10.0', 'v8.1'] version, but none were found
{% endhighlight %}

node.js 빌드 메뉴얼에 보니까 Windows 10 SDK 10.0.17763.0 or newer 가 필요하다고 한다
Visual Studio 2017을 업데이트 하면 SDK가 10.0.17763.0 같이 업데이트 된서 그냥 Visual Studio 2017를 마지막 버전으로 업데이트 했다
<br/>
다시 vcbuild.bat 파일을 실행

![img]({{ '/assets/images/build-node/node-1.JPG' | relative_url }}){: .center-image }

![img]({{ '/assets/images/build-node/node-2.JPG' | relative_url }}){: .center-image }

![img]({{ '/assets/images/build-node/node-3.JPG' | relative_url }}){: .center-image }

빌드 완료