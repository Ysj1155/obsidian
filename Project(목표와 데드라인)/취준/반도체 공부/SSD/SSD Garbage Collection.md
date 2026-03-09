# Garbage Collection이란 무엇인가
# 왜 필요한가
## 페이지 덮어쓰기 불가
## 블록 단위 erase
# 동작 흐름
## valid page 이동
## free block 회수
# 성능 영향
## tail latency
## steady-state
## WAF
# 관련 개념
## TRIM
## Over-Provisioning
## Preconditioning
# 검증 관점 질문
## steady-state 조건
## random write tail latency
## GC active 시 QoS
