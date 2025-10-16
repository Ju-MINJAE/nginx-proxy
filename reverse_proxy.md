## Reverse Proxy 

클라이언트가 Reverse Proxy에 요청하면,
Reverse Proxy가 관련 요청에 따라 적절한 내부 서버에 접속하여 결과를 받은 후 클라이언트에 전달

### 설정 방법 (1) — `nginx.conf` 직접 수정

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

    server {
        listen       80;
        server_name  localhost;

        # Reverse Proxy 설정
        location /api/ { # 요청받을 URL 경로
            proxy_pass http://localhost:8080/api/; # 전달할 백엔드 서버 주소
            proxy_http_version 1.1;
            
            # 필수 헤더 설정
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Port $server_port;
        }

        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
    }
}
```

### 동작 방식

- **요청**: `http://localhost/api/users`
- **전달**: `http://localhost:8080/api/users`로 프록시
- **응답**: 백엔드에서 처리 후 결과를 클라이언트에 반환


### 설정 방법 (2) — `/etc/nginx/sites-available/default` 수정

```nginx
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header Host $http_host;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;		
		proxy_set_header X-NginX-Proxy true;

		proxy_pass http://<IP>:<Port>; # reverse_proxy 적용할 IP, Port
	}
}
```

### 설정 적용
```bash
# 설정 파일 검증
nginx -t

# 설정 적용 (재시작 없이)
nginx -s reload
```

### 주요 헤더 설명

| 헤더 | 설명 |
|------|------|
| `Host` | 원본 요청의 호스트명 |
| `X-Real-IP` | 클라이언트의 실제 IP 주소 |
| `X-Forwarded-For` | 프록시 체인을 거친 모든 IP 주소 |
| `X-Forwarded-Proto` | 원본 요청의 프로토콜 (http/https) |
| `X-Forwarded-Port` | 원본 요청의 포트 번호 |

### 주의사항

- `$host`, `$remote_addr` 등은 Nginx 내장 변수이므로 그대로 사용
- `/api/` 경로는 프로젝트에 맞게 변경 가능
- `8080` 포트는 Spring Boot 실행 포트에 맞게 수정
