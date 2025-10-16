## Forward Proxy 

클라이언트 요청이 대상 서버에 직접 전송되지 않고 중간의 Proxy 서버를 통해 간접적으로 요청하고 응답을 받아오는 방식

Forward Proxy를 통해 정해진 사이트만 연결하도록 제한 가능 → 보안이 중요한 환경에서 사용

### 설정 방법

`nginx.conf` 파일 `http` 블럭 수정

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    # Forward Proxy 서버 설정
    server {
        listen       8888;  # Forward Proxy 포트
        server_name  localhost;

        # DNS 리졸버 설정 (필수)
        resolver 8.8.8.8 8.8.4.4;  # Google DNS

        # 접근 제어 (특정 IP만 허용)
        allow 192.168.1.0/24;  # 내부 네트워크 허용
        deny all;              # 나머지 차단
        
        # Forward Proxy 활성화
        location / {
            proxy_pass $scheme://$host$request_uri;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### 동작 방식

```
클라이언트 (192.168.1.100)
    ↓ (1) https://example.com 요청
Forward Proxy (Nginx:8888)
    ↓ (2) 대신 요청 전달
목적지 서버 (example.com)
    ↓ (3) 응답
Forward Proxy (Nginx:8888)
    ↓ (4) 응답 전달
클라이언트 (192.168.1.100)
```

> 목적지 서버는 Forward Proxy의 IP만 확인 가능 (클라이언트 IP 숨김)