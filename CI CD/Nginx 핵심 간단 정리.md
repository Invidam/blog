# 도입

프로젝트에서 웹 서버로 사용하는 Nginx에 대해 복습 목적으로 중요한 내용들만 간단히 정리해보려고 한다.

내가 정리하려자 하는 것은 [Nginx 자체 위키](https://www.nginx.com/resources/wiki/)의 도입부에 있는 다음 문구이다.

> Nginx는 **고성능 HTTP 서버**이고 **리버스 프록시**이다. **C10K 문제** 해결을 위해 등장하였고, **이벤트 기반 (비동기) 아키텍처**를 사용한다.

## C10K 문제

서버가 동시에(Concurrency) 10,000(10K)개의 클라이언트를 처리할 수 있냐는 문제이다.

### 기존의 웹 서버들의 한계

기존의 웹 서버들은 하나의 요청을 하나의 프로세스 혹은 쓰레드로 처리하여, 동시에 여러 요청을 처리하기에는 비효율적이었다. (C10K 해결 X)

Nginx는 하나의 쓰레드(Worker Thread)가 여러 요청을 비동기적으로 처리할 수 있어 C10K 문제를 해결할 수 있다.

[##_Image|kage@bB6iUe/btrUxoFWGFe/1vWew1au9q1kXMUfKLDSD1/img.png|CDM|1.3|{"originWidth":1024,"originHeight":821,"style":"alignCenter"}_##]

### Nginx 구조

[##_Image|kage@zQRrB/btrUusJldqn/cFZo4Yj4SYZx3ye20Vfsrk/img.png|CDM|1.3|{"originWidth":740,"originHeight":439,"style":"alignCenter"}_##]

- Master Process: 권한이 필요한 작업(설정 읽기, 포트 바인딩 등)을 진행한 후 아래 프로세스 들을 생성한다.
- Cache Loader: 실행된 후 디스크 기반의 캐시를 메모리에 로드한 후 종료된다.
- Cache Manager Process: 주기적으로 실행되며, 디스크 캐시의 메모리를 일정 크기 이내로 유지한다.
- Worker Process: 위 언급된 기능들 이외 모든 역할을 담당한다. (네트워크 연결, 디스크에 컨텐츠 읽기 & 쓰기, 업스트림 서버 통신)

## 이벤트 기반 (비동기) 아키텍처

Http Request에 의해 발생되는 **이벤트**를 읽은 뒤 Worker Process가 처리하며, Thread Pool을 이용하여 **비동기**적으로 동작한다.

자세한 설명은 아래를 참고하자.

### 동작 원리

**Nginx Event Loop**

Nginx는 이벤트 핸들러로 동작한다.  
즉, 커널로부터 커넥션 관련 정보를 받아 OS에게 명령을 내린다.

[##_Image|kage@mbMIB/btrUEvxggXS/AhDiegv9aT9VOntYZ0gV20/img.png|CDM|1.3|{"originWidth":790,"originHeight":558,"style":"alignCenter"}_##]

위 그림과 같이, Worker Process가 커널로 부터 이벤트를 받아 하나하나씩 실행한다.

**Event Loop 방식의 문제점**

Worker Process가 작업시간이 긴 요청을 처리할 때, 뒤에 있는 작업들이 오랜 시간을 기다려야 한다.

**해결책**

Worker Process에 Thread Pool을 둬, 작업시간이 긴 작업은 Worker Process가 직접 처리하지 않고 Thread Pool에 맡긴다.

[##_Image|kage@bcjpYP/btrUvX3i4eN/0EwlCwAYXVuTkiBq5WfM1k/img.png|CDM|1.3|{"originWidth":1024,"originHeight":484,"style":"alignCenter"}_##]

## 기타 특징들

### HTTP Server

Web Server과 같은 말로, 클라이언트의 HTTP 요청에 대해 정적인 파일(HTML, CSS, JS, 문서, 이미지 등)을 반환해주는 서비스를 의미한다.

### 리버스 프록시

일반적인 프록시(Forward Proxy)가 아닌 Client ↔ Internet ↔ Reverse Proxy ↔ Server 의 구성으로 이루어져 있다.

**특징**

- Forward Proxy가 클라이언트를 보호하는 것과 반대로, 서버를 보호하는데 사용한다.
- 클라이언트는 프록시 서버와 통신하고, 실제 서버와는 통신하지 못해 서버가 어떤 처리를 하였는지 알지 못한다.

**Forward Proxy**

- Client ↔ Proxy ↔ Internet ↔ Server 의 구성
- 클라이언트와 시스템서버 사이에 위치한다.
- 보안, 캐시, 트래픽 차단 등의 기능을 수행한다.

### 참고

[NGINX Thread Pool](https://www.nginx.com/blog/thread-pools-boost-performance-9x/)

[NGINX Processes](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)

[NGINX event-driven](https://vanseodesign.com/web-design/nginx-web-server/)

[Nginx 홈페이지](https://www.nginx.com/resources/wiki/)

### 기타

[WorkerThread와 Thread Pool 참고하면 좋은 블로그](https://ssup2.github.io/theory_analysis/Nginx_Thread_Pool/)
