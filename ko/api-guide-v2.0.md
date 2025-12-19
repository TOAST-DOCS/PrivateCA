# API v2.0 가이드
**Management > Private CA > API v2.0 가이드**
NHN Cloud Private CA API를 사용하여 인증서를 프로그래밍 방식으로 관리할 수 있습니다.

## 기본 정보

### API Endpoint

| 리전 | 엔드포인트 |
| --- | --- |
| KR1 | https://pca.api.nhncloudservice.com |

### API 목록

| Method | URI | 설명 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{certId}/download | PEM 형식의 인증서를 다운로드합니다. |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl | CRL 정보(PEM, 발행/갱신 시간)를 조회합니다. |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/der | DER 형식의 CRL을 다운로드합니다. |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/pem | PEM 형식의 CRL을 다운로드합니다. |
| POST | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl | CRL을 수동으로 갱신합니다. |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/ocsp/{ocspRequestBase64} | Base64 인코딩된 OCSP 요청으로 인증서 상태를 조회합니다. |
| POST | /v2.0/appkeys/{appkey}/cas/{caId}/ocsp | DER 형식의 OCSP 요청으로 인증서 상태를 조회합니다. |

## 사전 준비하기

### 인증

모든 API 요청에는 다음과 같은 인증 헤더가 필요합니다.

```
X-NHN-Authorization: Bearer {access_token}
```

!!! tip "알아두기"
    - 인증 헤더에 필요한 인증 토큰에 대한 자세한 사항은 [여기](https://docs.nhncloud.com/ko/nhncloud/ko/public-api/api-authentication/)에서 자세히 확인하실 수 있습니다.
    - 앱키(appkey)는 콘솔에서 확인할 수 있으며, 모든 API 경로에 포함되어야 합니다.

### 권한 관리

Private CA API는 역할 기반 접근 제어(RBAC)를 사용하며, 다음과 같이 구분됩니다.

- **VIEWER**: 인증서 다운로드, CRL 조회 등 읽기 작업만 수행할 수 있습니다.
- **ADMIN**: CRL 수동 갱신 등 모든 관리 작업을 수행할 수 있습니다.
- **공개 엔드포인트**: CRL 다운로드(DER/PEM)와 OCSP API는 인증서 검증을 위해 인증 없이 접근 가능합니다.

### 인증서 형식

Private CA API에서 사용하는 주요 인증서 형식은 다음과 같습니다.

- **PEM(privacy enhanced mail)**: 텍스트 기반 인증서 형식으로, Base64로 인코딩되어 있으며 `-----BEGIN CERTIFICATE-----`로 시작합니다. 사람이 읽을 수 있고 편집이 쉽습니다.
- **DER(distinguished encoding rules)**: 바이너리 형식의 인증서로, PEM보다 파일 크기가 작고 효율적입니다. 주로 Java 애플리케이션에서 사용됩니다.

## 인증서 다운로드 API

### 인증서 다운로드

발급된 인증서를 PEM 형식으로 다운로드합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{certId}/download
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caId | Long | Y | 저장소 ID |
| certId | Long | Y | 인증서 ID |

**필요 권한**

- `VIEWER` 이상

#### 응답

**Response Headers**

- Content-Type: `application/x-pem-file`
- Content-Disposition: `attachment; filename={filename}.pem`

**Response Body**

인증서 데이터(PEM 형식)

## CRL API

CRL(certificate revocation list)은 특정 발급자가 발급한 인증서 중 폐기된 인증서의 목록을 제공하는 메커니즘입니다. 클라이언트는 CRL을 다운로드하여 인증서가 폐기되었는지 확인할 수 있습니다.

### CRL 정보 조회

특정 발급자의 CRL 정보를 조회합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caId | Long | Y | 저장소 ID |
| issuerCertId | Long | Y | 발급자 인증서 ID |

**필요 권한**

- `VIEWER` 이상

#### 응답

**Response Body**

```json
{
  "header": {
    "resultCode": 0,
    "resultMessage": "SUCCESS",
    "isSuccessful": true
  },
  "body": {
    "crlPem": "-----BEGIN X509 CRL-----\n...\n-----END X509 CRL-----",
    "thisUpdate": "2024-01-01 00:00:00",
    "nextUpdate": "2024-01-08 00:00:00"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| crlPem | String | CRL PEM 형식 데이터 |
| thisUpdate | LocalDateTime | CRL 발행 시간 |
| nextUpdate | LocalDateTime | 다음 CRL 예정 시간 |

### CRL 다운로드(DER 형식)

CRL을 DER(바이너리) 형식으로 다운로드합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/der
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caId | Long | Y | 저장소 ID |
| issuerCertId | Long | Y | 발급자 인증서 ID |

**필요 권한**

- 권한 체크 없음(공개 엔드포인트)

#### 응답

**Response Headers**

- Content-Type: `application/pkix-crl`
- Content-Disposition: `attachment; filename=crl_{caId}_{certId}.crl`

**Response Body**

CRL 데이터(DER 형식)

### CRL 다운로드(PEM 형식)

CRL을 PEM 형식으로 다운로드합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/pem
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caId | Long | Y | 저장소 ID |
| issuerCertId | Long | Y | 발급자 인증서 ID |

**필요 권한**

- 권한 체크 없음(공개 엔드포인트)

#### 응답

**Response Headers**

- Content-Type: `application/pkix-crl`
- Content-Disposition: `attachment; filename=crl_{caId}_{certId}.pem`

**Response Body**

CRL 데이터(PEM 형식)

### CRL 수동 갱신

CRL을 수동으로 갱신합니다.

#### 요청

```
POST /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caId | Long | Y | 저장소 ID |
| issuerCertId | Long | Y | 발급자 인증서 ID |

**필요 권한**

- `ADMIN`

#### 응답

**Response Body**

```json
{
  "header": {
    "resultCode": 0,
    "resultMessage": "SUCCESS",
    "isSuccessful": true
  },
  "body": true
}
```

## OCSP API

OCSP(online certificate status protocol)는 개별 인증서의 폐기 상태를 빠르게 확인할 수 있는 프로토콜입니다. CRL과 달리 전체 목록을 다운로드하지 않고 특정 인증서의 상태만 요청 시점에 조회할 수 있습니다.

### OCSP 상태 조회(GET)

Base64로 인코딩된 OCSP 요청을 처리합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/cas/{caId}/ocsp/{ocspRequestBase64}
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caId | Long | Y | 저장소 ID |
| ocspRequestBase64 | String | Y | Base64로 인코딩된 OCSP 요청 |

!!! danger "주의"
    OCSP 요청을 Base64로 인코딩할 때는 URL-safe 형태로 변환해야 합니다.

    - 표준 Base64 인코딩 후 다음 문자들을 변환합니다:
        - `+` → `-`(plus를 hyphen으로)
        - `/` → `_`(slash를 underscore로)
    - Padding 문자(`=`)는 제거합니다.
    - 예시:
        - 변환 전: `MEow/SDAwL+oGCC+sGAQUF/BzAh==`
        - 변환 후: `MEow_SDAwL-oGCC-sGAQUF_BzAh`(`+` → `-`, `/` → `_`, `=` 제거)

**필요 권한**

- 권한 체크 없음(공개 엔드포인트)

**요청 예시**

```sh
# OCSP 요청 생성 및 Base64 인코딩
OCSP_REQUEST=$(openssl ocsp -issuer ca.pem -cert cert.pem -reqout - | base64 -w 0)

# URL 인코딩된 요청 전송
curl -X GET "https://pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/cas/1/ocsp/${OCSP_REQUEST}"
```

#### 응답

**Response Headers**

- Content-Type: `application/ocsp-response`

**Response Body**

OCSP 응답(DER 형식)

### OCSP 상태 조회(POST)

DER 형식의 OCSP 요청을 처리합니다.

#### 요청

```
POST /v2.0/appkeys/{appkey}/cas/{caId}/ocsp
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caId | Long | Y | 저장소 ID |

**필요 권한**

- 권한 체크 없음(공개 엔드포인트)

**Request Headers**

- Content-Type: `application/ocsp-request`

**Request Body**

OCSP 요청(DER 형식)

**요청 예시**

```sh
# OCSP 요청 생성
openssl ocsp -issuer ca.pem -cert cert.pem -reqout ocsp-request.der

# OCSP 요청 전송
curl -X POST "https://pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/cas/1/ocsp" \
  -H "Content-Type: application/ocsp-request" \
  --data-binary @ocsp-request.der \
  -o ocsp-response.der

# OCSP 응답 확인
openssl ocsp -respin ocsp-response.der -text
```

#### 응답

**Response Headers**

- Content-Type: `application/ocsp-response`

**Response Body**

OCSP 응답(DER 형식)

## 문제 해결하기

### CRL이 갱신되지 않는 경우

1. 콘솔의 저장소 상세 정보에서 CRL이 활성화되었는지 확인합니다.
2. 수동 갱신 API를 호출하여 즉시 갱신합니다.
3. CRL 갱신 주기(`crlRefreshPeriod`)를 확인하고 조정합니다.

### OCSP 응답이 없는 경우

1. 콘솔의 저장소 상세 정보에서 OCSP가 활성화되었는지 확인합니다.
2. 올바른 저장소 ID를 사용하는지 확인합니다.
3. OCSP 요청이 올바른 형식(DER)인지 확인합니다.

### OCSP 응답 결과가 실제 인증서 상태와 다른 경우

1. OCSP 응답은 갱신 주기에 따라 캐싱되므로, 최근에 인증서를 폐기한 경우 갱신 주기가 지날 때까지 이전 상태가 반환될 수 있습니다.
2. 콘솔의 저장소 상세 정보에서 OCSP 갱신 주기를 확인합니다.
3. 즉시 최신 상태를 확인해야 하는 경우, 갱신 주기가 경과한 후 다시 조회합니다.

### 인증서에 CRL/OCSP URL이 없는 경우

- 인증서 발급 시 해당 Extension이 포함되지 않은 경우입니다.
- 저장소 설정에서 CRL/OCSP를 활성화한 후 인증서를 재발급해야 합니다.

!!! danger "주의"
    - CRL/OCSP URL은 인증서 발급 시점에 포함되므로, 설정 변경 후에는 인증서를 재발급해야 합니다.
    - 이미 발급된 인증서의 CRL/OCSP URL은 변경할 수 없습니다.
