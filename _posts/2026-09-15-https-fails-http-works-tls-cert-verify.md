---
title: "연결이 비공개로 설정되어 있지 않습니다 — 인증서 검증 실패가 TLS handshake를 중단시키는 이유"
description: "HTTP는 되고 HTTPS만 안 됐다. handshake가 시작조차 못 한 게 아니라, 진행되던 handshake를 클라이언트가 중단시킨 것이었다."
date: 2026-09-15 00:00:58 +0900
categories: [트러블슈팅, 네트워크]
tags: [tls, https, letsencrypt, certbot, nginx, openssl, aws-ec2]
error_signatures:
  - "연결이 비공개로 설정되어 있지 않습니다"
  - "이 사이트는 안전하지 않습니다"
  - "Verify return code: 10 (certificate has expired)"
  - "Verify return code: 21 (unable to verify the first certificate)"
verified: 2026-09-15
---

- 배포한 서버에 `https://`로 접속하면 크롬이 "이 사이트는 안전하지 않습니다 / 연결이 비공개로 설정되어 있지 않습니다"를 띄우고, 같은 서버에 `http://`로는 정상 접속됐다.
- HTTP는 되고 HTTPS만 안 된다는 건 TLS handshake가 애초에 시작조차 안 된 게 아니라, **handshake가 진행되다가 클라이언트의 인증서 검증 단계에서 중단(abort)됐다**는 신호다.
- 해결은 인증서 재발급으로 끝났다. 다만 **검증이 실패한 정확한 사유는 확정하지 못했다** — 아래 3-4에 그대로 남겨뒀다.

---

## 1. 문제 상황

팀 프로젝트로 진행 중인 PACE 서버(Android 앱 + Spring Boot 백엔드)를 AWS EC2에 배포하고 도메인(`pace-server.kro.kr`)을 연결한 뒤, 브라우저로 접속하니 다음 화면이 떴다.

> **연결이 비공개로 설정되어 있지 않습니다**
> 공격자가 pace-server.kro.kr에서 비밀번호, 메시지 또는 신용카드 정보를 훔치려고 시도하고 있을 수 있습니다.

이상했던 점은, 주소창에서 프로토콜만 `https://`에서 `http://`로 바꾸면 정상 접속이 됐다는 것이다.

| 요청 | 결과 |
| --- | --- |
| `https://pace-server.kro.kr` | 접속 실패 (인증서 경고) |
| `http://pace-server.kro.kr` | 접속 성공 |

이 차이가 원인을 좁히는 데 가장 중요한 단서였다.

## 2. 환경

- **인프라**: AWS EC2 (Amazon Linux 2023, t2.micro), Elastic IP로 고정된 퍼블릭 IP
- **컨테이너**: Docker Compose로 nginx / MySQL / Redis 구성
- **도메인**: `pace-server.kro.kr` (무료 서브도메인 서비스, 위 Elastic IP를 가리키도록 설정)
- **인증서**: Let's Encrypt로 발급, nginx에서 종단(termination) 처리
- 확인 시점: 2026-09-15

EIP는 이번 문제의 직접 원인은 아니지만, 도메인이 인스턴스 재시작 여부와 무관하게 항상 같은 IP를 가리키게 해주는 전제 조건이라 환경에 포함해둔다. (EIP가 없으면 인스턴스를 재시작할 때마다 퍼블릭 IP가 바뀌어 도메인 매핑이 깨지는, 이번과는 또 다른 장애가 생긴다.)

## 3. 원인을 좁혀나간 과정

### 3-1. 오진 ①: 범위를 너무 넓게 잡았다

맨 처음에는 "SSL/TLS 인증서 문제인가?" 싶어서 SSL/TLS 스펙 자체를 공식문서로 찾아보기 시작했다. 파다 보니 HTTP/HTTPS 프로토콜 전반으로 범위가 계속 넓어졌다.

**왜 틀렸나.** 지금 겪은 증상을 설명하는 데 필요한 개념보다 훨씬 넓은 범위를 공부하고 있었다. 증상이 이미 범위를 좁혀주고 있었는데 그걸 안 썼다.

### 3-2. 증상으로 범위를 좁히기

정리된 흐름은 이랬다.

1. 브라우저는 기본적으로 `https`로 먼저 접속을 시도한다.
2. 서버 쪽 인증서에 문제가 있으면 프로토콜(TLS) 단계에서 막힌다.
3. `http`로는 접속이 된다 → 즉 **서버가 죽은 것도, 네트워크가 막힌 것도 아니고, TLS 계층에서만 실패**하고 있다.

이 지점에서 "인증서 파일 자체가 없거나, 있어도 유효하지 않은 상태"로 원인 후보가 좁혀졌다. 이후 서버를 직접 관리하는 팀원이 인증서를 재발급하면서 증상이 사라졌다.

### 3-3. 오진 ②: "handshake 자체가 안 이뤄진다"

처음에는 "인증서가 유효하지 않으면 TLS handshake 자체가 안 이뤄진다"라고 이해했다.

**왜 틀렸나.** 정확히는 이렇다.

> **인증서 검증 실패는 handshake가 시작되지 않는 게 아니라, 진행되던 handshake를 클라이언트가 중간에 중단(abort)시키는 것이다.**

이 차이가 왜 중요한지는 4번에서 handshake 순서를 짚으면 명확해진다.

### 3-4. 확정하지 못한 것

> 검증이 실패한 **정확한 사유**(만료 / 체인 누락 / 도메인 불일치)는 내가 직접 확인하지 못했다. 팀원이 재발급하면서 증상이 사라졌고, 그 시점의 `openssl s_client` 출력과 `Verify return code`를 기록해두지 않았다. 5번은 "다음에 같은 증상을 만나면 이렇게 확정한다"는 절차이고, **이번 건의 실측값이 아니다.**
{: .prompt-warning }

그래도 남아 있는 사실로 후보를 좁힐 수는 있다. 이 인증서는 **7월 10일에 최초 발급**된 것으로 확인됐다. Let's Encrypt 인증서의 유효기간은 90일이므로 만료는 **10월 8일 전후**다. 증상을 본 9월 15일은 그보다 3주 이상 이전이다.

**즉 만료였을 가능성은 낮다.** 그렇다면 남는 후보는

- nginx에 `fullchain.pem` 대신 `cert.pem`을 지정해서 **중간 인증서가 빠진 경우** (`Verify return code: 21`)
- 인증서의 도메인(CN/SAN)이 실제 접속 도메인과 다른 경우

쪽이 더 유력하다. 다만 **확인하지 못했으므로 추정으로 남겨둔다.** 이게 이 글의 가장 큰 구멍이고, 재발 방지 항목을 "만료 자동 갱신"만으로 끝내면 안 되는 이유다.

## 4. TLS Handshake는 어떻게 진행되고, 왜 중단되는가

TLS 1.2 기준 흐름이다.

```text
Client                                          Server
  |------------- ClientHello ------------------>|
  |<--- ServerHello, Certificate, ServerHelloDone
  |  (여기서 클라이언트가 인증서를 검증)
  |
  |  [검증 성공]
  |------------- ClientKeyExchange, Finished -->|
  |<------------------------- Finished ----------
  |============ 암호화된 데이터 통신 시작 ============
  |
  |  [검증 실패]
  |------------- Alert (fatal, bad_certificate) >|
  |============ 연결 종료 ============
```

핵심은 3단계다.

1. **ClientHello** — 클라이언트가 지원하는 TLS 버전, 암호 스위트 목록 등을 서버에 전달하며 handshake가 시작된다.
2. **ServerHello + Certificate** — 서버가 자신의 인증서를 클라이언트에게 보낸다. **여기까지는 인증서 상태와 무관하게 정상 진행된다.**
3. **클라이언트의 인증서 검증** — 클라이언트(브라우저)가 받은 인증서를 검사한다.
   - 유효기간이 지나지 않았는가 (`Not After`)
   - 인증서의 도메인(CN/SAN)이 실제 접속 도메인과 일치하는가
   - 인증서 체인이 신뢰할 수 있는 루트 CA까지 이어지는가 (중간 인증서 누락 여부)

이 중 하나라도 실패하면 클라이언트는 **fatal alert**(`certificate_expired`, `bad_certificate`, `unknown_ca` 등)를 서버에 보내고 스스로 연결을 끊는다. 서버와 통신 자체는 이미 하고 있었지만, "이 인증서를 신뢰할 수 없다"는 판단을 클라이언트가 내리는 순간 handshake가 미완성 상태로 종료된다.

브라우저가 "연결이 비공개로 설정되어 있지 않습니다"를 보여주는 시점이 바로 이 단계다 — handshake가 완료되지 않아 암호화된 채널이 만들어지지 못했으므로, 브라우저는 안전을 보장할 수 없는 연결이라고 판단하고 경고를 띄운다.

## 5. 직접 확인하는 법

브라우저 대신 커맨드라인으로 handshake 로그를 직접 보는 게 가장 빠르다.

```bash
# TLS handshake 과정을 상세히 출력
openssl s_client -connect pace-server.kro.kr:443 -servername pace-server.kro.kr
```

출력 마지막의 `Verify return code`가 실패 사유를 그대로 알려준다.

```text
Verify return code: 10 (certificate has expired)
```

```text
Verify return code: 21 (unable to verify the first certificate)
```

`10`은 만료, `21`은 **체인이 끊긴 것**(중간 인증서 누락)이다. 증상 화면은 똑같지만 원인과 해결이 다르므로, 이 숫자를 보지 않고 넘어가면 안 된다.

curl로도 확인할 수 있다.

```bash
curl -v https://pace-server.kro.kr
```

서버에 접속할 수 있다면 인증서의 유효기간과 nginx가 실제로 어떤 파일을 쓰고 있는지도 본다.

```bash
sudo certbot certificates

openssl x509 -in /etc/letsencrypt/live/pace-server.kro.kr/fullchain.pem -noout -dates

# nginx가 fullchain이 아니라 cert를 가리키고 있지는 않은지
sudo grep -r ssl_certificate /etc/nginx/
```

## 6. 해결

Let's Encrypt 인증서를 재발급/갱신하는 것으로 해결됐다.

```bash
sudo certbot renew
sudo systemctl reload nginx
```

재발급 후 다시 `openssl s_client`로 확인했을 때 `Verify return code: 0 (ok)`가 나오면 정상이다.

## 7. 재발 방지

**① 자동 갱신을 걸어둔다.** Let's Encrypt 인증서는 90일마다 만료되므로 수동 갱신은 언젠가 놓친다.

```bash
# 갱신 가능 여부만 테스트 (실제 갱신은 하지 않음)
sudo certbot renew --dry-run

# systemd timer 상태 확인 (설치 방식에 따라 이미 등록된 경우가 많다)
systemctl list-timers | grep certbot
```

등록되어 있지 않다면 cron으로 하루 1~2회 갱신 체크를 걸어둔다.

```bash
0 3 * * * certbot renew --quiet && systemctl reload nginx
```

**② nginx가 `fullchain.pem`을 가리키는지 확인한다.** 이번 건에서 만료가 아닐 가능성이 큰 만큼, 자동 갱신만으로는 재발을 막지 못한다. `cert.pem`을 지정하면 갱신이 아무리 잘 돌아도 체인이 끊긴 상태가 유지된다.

```nginx
ssl_certificate     /etc/letsencrypt/live/도메인/fullchain.pem;  # cert.pem 아님
ssl_certificate_key /etc/letsencrypt/live/도메인/privkey.pem;
```

**③ 증상이 사라지기 전에 `Verify return code`를 먼저 찍는다.** 이번에 못 한 것이 이거다. 고치면 증거가 같이 사라진다.

## 8. 정리

| | |
| --- | --- |
| 증상 | HTTPS만 접속 실패, HTTP는 정상 → TLS 계층에서만 문제라는 강한 신호 |
| 메커니즘 | 인증서 검증 실패로 handshake가 **중단**됨 (시작되지 않은 게 아니다) |
| 근본 원인 | **미확정.** 발급일 역산상 만료는 아닐 가능성이 큼 → 체인 누락이 유력하나 미확인 |
| 해결 | 인증서 재발급 |
| 재발 방지 | 자동 갱신 + `fullchain.pem` 확인 + 고치기 전에 `Verify return code` 기록 |

**설명** — HTTPS만 실패하는 이유는, 서버가 인증서를 보내는 2단계까지는 인증서 상태와 무관하게 진행되고, 3단계에서 클라이언트가 검증에 실패하면 스스로 fatal alert를 보내 연결을 끊기 때문이다. 서버는 멀쩡하다.

**예측** — 같은 인증서로 `curl -k`를 붙이면? 검증을 건너뛰므로 **성공한다.** 이게 "서버가 죽은 게 아니다"의 가장 빠른 증명이다.

**파괴** — nginx의 `ssl_certificate`를 `fullchain.pem`에서 `cert.pem`으로 바꾸면 브라우저 경고는 같지만 `Verify return code`가 `21`로 바뀐다. 반대로 시스템 시간을 만료 이후로 당기면 `10`이 된다. **증상 화면은 같은데 코드가 다르다는 게, 이 숫자를 봐야 하는 이유다.**

이번 건에서 얻은 교훈은, 증상(HTTP는 되고 HTTPS만 안 됨)을 먼저 정확히 관찰하면 "TLS 계층 문제"까지는 스펙을 몰라도 추론할 수 있다는 것이었다. 그 다음에 필요한 개념(handshake의 인증서 검증 단계)만 좁혀서 보니, 처음처럼 SSL/TLS와 HTTP/HTTPS 전체를 훑는 것보다 훨씬 빨랐다. 반대로 못 한 것은 **고치기 전에 증거를 남기는 것**이었다.
