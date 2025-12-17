# Management > Private CA > ACME 인증서 갱신 가이드
Private CA 서비스는 ACME(Automatic Certificate Management Environment) 프로토콜을 지원하여 인증서의 자동 발급 및 갱신을 가능하게 합니다. ACME 클라이언트(예: Certbot)를 사용하면 수동 개입 없이 인증서를 발급받고 주기적으로 갱신할 수 있습니다.

본 가이드에서는 Private CA ACME 서버를 이용하여 Certbot으로 인증서를 발급하는 방법을 안내합니다.

!!! tip "알아두기"
    - **ACME (Automatic Certificate Management Environment)**: 인증서 발급과 갱신을 자동화하는 표준 프로토콜입니다.
    - **Certbot**: Let's Encrypt에서 개발한 ACME 클라이언트 도구로, 인증서를 자동으로 발급하고 갱신합니다.
    - **Base 인증서**: ACME 자동 갱신 시 참조하는 템플릿 역할의 인증서입니다.
    - **CSR (Certificate Signing Request)**: 인증서 발급을 요청하기 위한 서명 요청 파일입니다.
    - **EAB (External Account Binding)**: ACME 서버에 인증하기 위한 계정 바인딩 정보입니다.

## 사전 준비

ACME를 통한 인증서 발급을 시작하기 전에 다음 사항을 준비해야 합니다.

### 1. Base 인증서 발급

Base 인증서는 ACME 서버가 자동 갱신 시 참조하는 "템플릿" 역할을 합니다.

- Base 인증서에 설정된 도메인(CN, SAN)과 동일한 도메인의 인증서만 ACME를 통해 갱신할 수 있습니다.
- Base 인증서는 콘솔에서 일반적인 인증서 발급 절차로 생성합니다.
- Base 인증서를 발급받은 후, 해당 인증서의 ID를 ACME Directory URL에 사용합니다.

### 2. Certbot 설치

Certbot은 가장 널리 사용되는 ACME 클라이언트입니다. [Certbot 공식 문서](https://certbot.eff.org/)를 참고하여 운영 체제에 맞게 설치합니다.

**설치 예시 (Ubuntu/Debian):**

```bash
sudo apt update
sudo apt install certbot
```

**설치 예시 (CentOS/RHEL):**

```bash
sudo yum install certbot
```

### 3. ACME 서버 정보 확인

Private CA 콘솔에서 다음 정보를 확인합니다.

- **ACME Directory URL**: `https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory`
- **ACME 토큰 ID**: 콘솔에서 발급한 ACME 토큰 ID (**YOUR_ACME_TOKEN_ID**)
- **ACME HMAC 키**: 콘솔에서 발급한 ACME 토큰 HMAC 키 (**YOUR_ACME_TOKEN_HMAC_KEY**)

## 인증서 갱신하기

Certbot을 사용하여 인증서를 발급받는 절차는 다음과 같습니다.

### 명령어 구성

다음은 기본적인 인증서 발급 명령어 예시입니다.

```bash
certbot certonly \
  --manual \
  --manual-auth-hook ./pre.sh \
  --deploy-hook ./post.sh \
  --preferred-challenges http \
  --server https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory \
  -d example.com -d www.example.com \
  --eab-kid "YOUR_ACME_TOKEN_ID" \
  --eab-hmac-key "YOUR_ACME_TOKEN_HMAC_KEY" \
  --key-type rsa \
  --rsa-key-size 2048 \
  --agree-tos \
  --register-unsafely-without-email
```

### 주요 옵션 설명

| 옵션 | 설명 | 필수 | 기본값 |
|------|------|------|--------|
| `--manual` | Challenge 검증을 수동으로 수행합니다. Apache나 Nginx 자동 설정 대신 수동 모드를 사용합니다. | O | - |
| `--manual-auth-hook` | Challenge 검증 전에 실행할 스크립트를 지정합니다. 빈 스크립트라도 renew 시 오류 방지를 위해 필요합니다. | O | - |
| `--deploy-hook` | 인증서 발급 성공 후 실행할 스크립트를 지정합니다. 인증서 이동, 배포 등의 후처리에 사용합니다. | X | - |
| `--preferred-challenges` | Challenge 방식을 지정합니다 (`http`, `dns`, `tls-alpn`). 일반적으로 `http`를 사용합니다. | O | - |
| `--server` | ACME 서버의 Directory URL을 지정합니다. | O | - |
| `-d` | 인증서에 포함할 도메인을 지정합니다. Base 인증서의 CN과 SAN을 콘솔에서 확인하여 정확히 입력해야 합니다. | O | - |
| `--eab-kid` | External Account Binding의 키 ID입니다. Private CA에서 발급한 ACME 토큰 ID를 입력합니다. | O | - |
| `--eab-hmac-key` | EAB HMAC 키 (Base64 인코딩)입니다. Private CA에서 발급한 ACME 토큰 HMAC 키를 입력합니다. | O | - |
| `--key-type` | 개인키 유형 (`rsa`, `ecdsa`)을 지정합니다. | X | rsa |
| `--rsa-key-size` | RSA 키 길이를 지정합니다. `--key-type rsa`일 경우에만 사용하며, `2048`, `3072`, `4096` 값을 사용할 수 있습니다. | X | 2048 |
| `--elliptic-curve` | ECDSA 키 곡선을 지정합니다. `--key-type ecdsa`일 경우에만 사용하며, `secp256r1`, `secp384r1`, `secp521r1` 값을 사용할 수 있습니다. | X | secp256r1 |
| `--agree-tos` | 서비스 약관에 자동으로 동의합니다. 인터랙티브 입력을 방지합니다. | X | - |
| `--register-unsafely-without-email` | 이메일 없이 계정을 등록합니다. 인터랙티브 입력을 방지합니다. | X | - |

!!! tip "알아두기"
    - `--manual` 모드는 수동 Challenge 검증에 사용됩니다.
    - `manual-auth-hook`은 인증서 renew 시 안정성 확보를 위해 반드시 필요합니다.
    - `deploy-hook`을 활용하면 인증서 발급 후 자동으로 배포 및 후처리 작업을 수행할 수 있습니다.
    - `--server`, `--eab-kid`, `--eab-hmac-key`는 Private CA ACME 서버 연동에 필수입니다.

!!! danger "주의"
    도메인 지정 시 Base 인증서에 설정된 CN(Common Name)과 도메인 SAN(Subject Alternative Name)을 정확히 입력해야 합니다. 인증서 발급 전 콘솔에서 Base 인증서의 CN과 SAN 정보를 확인하여 `-d` 옵션에 올바른 도메인을 지정했는지 반드시 검증하십시오.

### Hook 스크립트 예시

#### pre.sh (인증 전 실행 스크립트)

`manual-auth-hook`은 내용이 비어 있어도 파일로 존재해야 합니다. 이는 인증서 갱신(renew) 시 오류를 방지하기 위함입니다.

```bash
#!/bin/bash
# 필요한 인증 전처리 작업을 여기에 추가할 수 있습니다.
```

#### post.sh (인증서 발급 후 실행 스크립트)

`deploy-hook`은 인증서가 성공적으로 발급된 경우에만 실행됩니다.

```bash
#!/bin/bash
# 인증서 발급 완료 후 처리 작업
echo "[post.sh] 인증서 발급 완료!"

# 인증서를 원하는 위치로 복사
cp /etc/letsencrypt/live/example.com/fullchain.pem ~/Downloads/
cp /etc/letsencrypt/live/example.com/privkey.pem ~/Downloads/

# 웹 서버 재시작 등 추가 작업
# systemctl reload nginx
```

## 발급된 인증서 확인

인증서는 기본적으로 다음 경로에 저장됩니다.

```
/etc/letsencrypt/live/<도메인명(CN)>/
├── cert.pem          # 서버 인증서
├── chain.pem         # 중간 인증서 체인
├── fullchain.pem     # cert.pem + chain.pem
└── privkey.pem       # 개인키
```

### 인증서 내용 확인

```bash
# 인증서 정보 확인
openssl x509 -in /etc/letsencrypt/live/<도메인명(CN)>/cert.pem -text -noout

# 인증서 유효기간 확인
openssl x509 -in /etc/letsencrypt/live/<도메인명(CN)>/cert.pem -noout -dates
```

## 인증서 자동 갱신

Certbot은 만료가 임박한 인증서를 자동으로 갱신할 수 있습니다.

### 갱신 전제 조건

- 최초 발급 이력이 있어야 합니다.
- `/etc/letsencrypt/renewal/<도메인>.conf` 파일이 존재해야 합니다.
- `/etc/letsencrypt/live/<도메인>/` 디렉터리에 인증서 파일들이 있어야 합니다.

### 자동 갱신 설정

Certbot 설치 시 자동으로 cron 또는 systemd timer가 등록되어 주기적으로 인증서 만료 여부를 확인합니다.

**기본 갱신 주기**: 만료 30일 전부터 자동 갱신 시도

### 수동 갱신

필요 시 다음 명령어로 수동 갱신을 수행할 수 있습니다.

```bash
# 모든 인증서 갱신 시도
sudo certbot renew

# 강제 갱신
sudo certbot renew --force-renewal
```

### Cron 작업 등록

자동 갱신이 등록되지 않았다면 수동으로 cron 작업을 추가할 수 있습니다.

```bash
# crontab 편집
sudo crontab -e

# 매일 새벽 2시에 갱신 확인
0 2 * * * certbot renew --no-random-sleep-on-renew
```

### 갱신 주기 변경

`/etc/letsencrypt/renewal/<도메인>.conf` 파일에서 갱신 주기를 조정할 수 있습니다.

```ini
[renewalparams]
renew_before_expiry = 30 days
```

`renew_before_expiry` 값을 변경하여 인증서 만료 며칠 전부터 갱신을 시도할지 설정할 수 있습니다.

!!! danger "주의"
    - ED25519 키 타입은 현재 Certbot에서 지원하지 않습니다. 인증서 체인에 ED25519 알고리즘을 사용하는 인증서가 포함된 경우 발급 및 갱신이 불가능합니다.
    - 인증서 체인은 최소 3단 이상(Root → Intermediate → Leaf) 구성되어야 합니다.
    - `renew_before_expiry` 값은 ACME 서버의 Rate Limit 정책을 고려하여 신중히 조정해야 합니다.
    - Certbot 설치 시 자동 등록된 cron 작업에는 기본 옵션이 포함되어 있을 수 있으므로, 필요에 따라 `/etc/cron.d/certbot` 파일을 수정하는 것이 권장됩니다.
    - 기본 cron 작업에는 랜덤 지연(`perl -e 'sleep int(rand(43200))'`)이 포함되어 있습니다. 이는 ACME 서버 과부하 방지를 위한 것으로, 즉시 실행이 필요하면 해당 구문을 제거하거나 `--no-random-sleep-on-renew` 옵션을 사용해야 합니다.

## 문제 해결

### 인증서 발급 실패 시

1. **ACME Directory URL 확인**: `--server` 옵션의 URL이 올바른지 확인합니다.
2. **EAB 인증 정보 확인**: `--eab-kid`와 `--eab-hmac-key` 값이 정확한지 확인합니다.
3. **도메인 검증 실패**: Challenge 방식이 환경에 맞는지 확인합니다. HTTP Challenge의 경우 80번 포트가 열려 있어야 합니다.
4. **Hook 스크립트 권한**: `pre.sh`와 `post.sh` 파일에 실행 권한이 있는지 확인합니다.

### 인증서 갱신 실패 시

1. **Renewal 설정 확인**: `/etc/letsencrypt/renewal/<도메인>.conf` 파일이 존재하고 올바른지 확인합니다.
2. **Hook 스크립트 존재 확인**: `manual-auth-hook`으로 지정한 스크립트가 여전히 존재하는지 확인합니다.

## ACME 프로토콜 정보

Private CA에서 제공하는 ACME Directory URL(`/directory`)을 통해 ACME 클라이언트는 필요한 모든 엔드포인트 정보를 자동으로 가져옵니다.

ACME 프로토콜의 전체 흐름은 클라이언트에 의해 자동으로 처리되므로, 사용자는 Directory URL만 제공하면 됩니다.

ACME 프로토콜에 대한 자세한 내용은 [RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555)를 참고하십시오.

## 참고 자료

- [Certbot 공식 문서](https://certbot.eff.org/)
- [ACME 프로토콜 명세 (RFC 8555)](https://datatracker.ietf.org/doc/html/rfc8555)
- [Let's Encrypt - Challenge Types](https://letsencrypt.org/docs/challenge-types/)
