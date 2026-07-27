## 보안 > DDoS Guard > L7 DDoS 보안 설정 가이드

여기에서는 L7 DDoS 공격을 지원하는 보안 설정 방법을 설명합니다.

## Nginx

| 번호 | 항목 | 설정 방법 | 내용 | 우선 순위 | 예시 |
| --- | --- | --- | ---- | ---- | ---- |
| 1 | 요청 속도 제한(Rate Limit) | limit_req_zone /limit_req 설정 | IP별 요청 수 제한으로 HTTP 요청을 하도록 | 필수 | http {< BR >limit_req_zone $binary_remote_addr zone=req_limit_per_ip:10m rate=5r/s; < BR >} 서버 {< BR >limit_req zone=req_limit_per_ipburst=10 nodelay; < BR >} |
| 2 | 장면 연결 제한(Connection Limit) | limit_conn_zone /limit_conn 설정 | 하나의 IP에서 연결을 맺을 수 있는 연결 수 제한 | 필수 | http {< BR >limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m; < BR >} 서버 {< BR >limit_conn conn_limit_per_ip 10; < BR >} |
| 3 | 인용 본문 크기 제한 | client_max_body_size 설정 | 디스플레이 POST로 교체하기 | 필수 | client_max_body_size 1m; |
| 4 | 백업 크기 제한 | client_body_buffer_size, client_header_buffer_size 설정 | 요청 헤더·본문 주차를 제한(Slowloris를 공격하는 것) | 필수 | client_body_buffer_size 16k; < BR >client_header_buffer_size 1k; |
| 5 | 연결 유지 제한 | keepalive_timeout 설정 | 클라이언트의 세션 점유 시간 제한 | 필수 | keepalive_timeout 10초; |
| 6 | 요청 시간 제한 | client_header_timeout, send_timeout 설정 | 느린 HTTP(Slow HTTP) 공격을 요청합니다 | 필수 | client_header_timeout 10초; < BR >send_timeout 10초; |
| 7 | HTTP 방식 제한 | if문으로 허용된 방법만 제한 | 처리 방법(TRACE, PUT 등) 요청을 차단 | 주의사항 | if ($request_method ! ~ ^(GET \| POST \| HEAD)$) { return 444; } |
| 8 | 비정상 User-Agent 차단 | 정규식으로 User-Agent 시리즈 | 네, 봇, 컬 등 자동화 툴 접근 차단 | 주의사항 | if ($http_user_agent ~ * (masscan \| 컬 \| python \| nmap)) { return 403; } |
| 9 | 상태 모니터링 | stub_status 설정 | 매주 요청/세션 수확인(운영 점검용) | 주의사항 | 위치 /nginx_status {< BR > stub_status;< BR > 127.0.0.1 허용;< BR > 모두 거부; < BR >} |
| 10 | 캐싱 설정 | Proxy_cache 설정 | 같은 요청 캐싱으로 백엔드의 설명 | 주의사항 | Proxy_cache_path /tmp/nginx_cache 레벨=1:2key_zone=my_cache:10m; < BR >위치 / {< BR > Proxy_cache my_cache;< BR > Proxy_cache_use_stale 오류 시간 초과 업데이트 중; < BR >} |

## Apache

| 번호 | 항목 | 설정 방법 | 내용 | 우선 순위 | 예시 |
| --- | --- | --- | ---- | ---- | ---- |
| 1 | mod_evasive 설정 | yum install mod_evasive 후 /etc/httpd/conf.d/mod_evasive.conf 설정 | 짧은 시간 내 잊어버리기 IP 자동 차단 | 필수 | DOSPageCount 2 < BR >DOSSiteCount 50 < BR >DOSBlockingPeriod 10 |
| 2 | mod_qos 설정 | yum install mod_qos 후 /etc/httpd/conf.d/mod_qos.conf | IP별 최대 연결 수 및 요청 수 제한 | 필수 | QS_SrvMaxConnPerIP 10 < BR >QS_SrvMaxConnClose 20 < BR >QS_SrvRequestRate 5 |
| 3 | KeepAlive 제한 | KeepAliveTimeout 설정 | 거리감 유지 방지 | 필수 | KeepAlive 켜기 < BR >MaxKeepAliveRequests 100 < BR >KeepAliveTimeout 5 |
| 4 | 인용 본문 크기 제한 | LimitRequestBody 설정 | 디스플레이 POST 요청 제한 | 필수 | LimitRequestBody 1048576 |
| 5 | 타임아웃 조정 | Timeout, RequestReadTimeout 설정 | 계속 요청/응답 | 필수 | 시간 초과 10 < BR >RequestReadTimeout 헤더=10-20,MinRate=500 |
| 6 | HTTP 방식 제한 | <LimitExcept \> 블록 사용 | 허용된 방법만 허용 | 필수 | <LimitExcept GET POST HEAD \> < BR > 모두 거부 < BR ></LimitExcept \>  |
| 7 | 사용자-에이전트 부분 | SetEnvIfNoCase + 거부 | 비정상 User-Agent 차단 | 주의사항 | SetEnvIfNoCase 사용자 에이전트 "curl" bad_bot < BR >주문 허용, 거부 < BR >모두 허용 < BR >env=bad_bot에서 거부 |
| 8 | 속도 제한(mod_ratelimit) | mod_ratelimit 사용 | 응답 속도 제한으로 세션 | 주의사항 | SetOutputFilter RATE_LIMIT < BR >SetEnv 속도 제한 400 |
| 9 | 생일 축하 약속 | LogFormat 수정 | 요청, 응답 크기, User-Agent가 포함되어 추적성 강화 | 주의사항 | LogFormat "%h %l %u %t \\ "%r \\ " %>s %b \\ "%{Referer}i \\ " \\ "%{User-Agent}i \\ "" 결합 |

## 로드 밸런싱

| 번호 | 항목 | 설정 방법 | 내용 | 예시 |
| --- | --- | --- | ---- | ---- |
| 1 | 세션 연결 제한 | 연결 제한 설정 | 리스너가 동시에 TCP 세션의 수를 안내 | 60,000 < BR >서비스 방향에 따라 적절한 조정이 필요합니다 |
| 2 | 연결 유지 제한 | Keep-Alive 타임아웃 설정 | 클라이언트 및 서버와의 세션 유지 시간을 기본으로 공지 | 있었다: 300초 |
| 3 | 비정상적으로 연인 | 유효하지 않은 거부 설정 | HTTP 요청 헤더에 유효하지 않은 문자가 포함된 경우 | 사용: 사용 |
