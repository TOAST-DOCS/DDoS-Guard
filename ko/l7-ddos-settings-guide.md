## Security > DDoS Guard > L7 DDoS 보안 설정 가이드

여기에서는 L7 DDoS 공격을 효과적으로 대응하기 위한 보안 설정 방법을 설명합니다.

## Nginx

| 번호 | 항목 | 설정 방법 | 내용 | 예시 |
| --- | --- | --- | ---- | ---- |
| 1 | 요청 속도 제한 (Rate Limit) | limit_req_zone / limit_req 설정 | IP별 초당 요청 수 제한으로 과도한 HTTP 요청 방어 | http {<BR>   limit_req_zone $binary_remote_addr zone=req_limit_per_ip:10m rate=5r/s; <BR>} server {<BR>   limit_req zone=req_limit_per_ip burst=10 nodelay; <BR>} |
