## API v2.0 가이드
**Management > Private CA > API v2.0 가이드**

NHN Cloud Private CA API를 사용하여 인증서를 프로그래밍 방식으로 관리할 수 있습니다.

## Private CA API 공통 정보

### API 엔드포인트

| 리전 | 엔드포인트 |
| --- | --- |
| KR1 | https://kr1-pca.api.nhncloudservice.com |

### 인증 및 권한

Private CA API v2.0은 API 호출 및 인증을 위한 인증 방법으로 Appkey, User Access Key 토큰을 지원합니다.

Appkey는 API 호출 시 요청 URL에 포함하여 특정 리소스를 가리키고 식별하는 데 사용됩니다.
User Access Key 토큰은 User Access Key를 기반으로 발급되는 Bearer 타입의 일시적 액세스 토큰으로, API 호출 시 인증/인가를 위해 사용합니다.

각 인증 방법의 확인 및 사용에 대한 자세한 내용은 각각 [Appkey](/nhncloud/ko/public-api/appkey/), [User Access Key 토큰](/nhncloud/ko/public-api/user-access-key-token/)를 참고하세요.

### API 목록

#### 저장소

| Method | URI | 설명 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores | 저장소 목록을 조회 |
| POST | /v2.0/appkeys/{appkey}/ca-stores | 저장소를 생성 |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | 저장소 상세 정보를 조회 |
| PUT | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | 저장소를 수정 |
| DELETE | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | 저장소를 삭제 |

#### 인증서(발급자)

| Method | URI | 설명 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs | 인증서 목록을 조회 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs | 인증서(발급자)를 발급(ROOT, INTERMEDIATE만 발급 가능) |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId} | 인증서 상세 정보를 조회 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/revoke | 인증서를 폐기 |

#### 템플릿

| Method | URI | 설명 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates | 템플릿 목록을 조회 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates | 템플릿을 생성 |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId} | 템플릿 상세 정보를 조회 |
| PUT | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId} | 템플릿을 수정 |
| DELETE | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId} | 템플릿을 삭제 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}/certificates | 템플릿으로 인증서를 발급 |

#### 인증서 다운로드

| Method | URI | 설명 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/download | PEM 형식 인증서를 다운로드 |

#### CRL

| Method | URI | 설명 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl | CRL 정보(PEM, 발행/갱신 시간)를 조회 |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/der | DER 형식 CRL을 다운로드 |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/pem | PEM 형식 CRL을 다운로드 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl | CRL을 수동으로 갱신 |

#### OCSP

| Method | URI | 설명 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp/{ocspRequestBase64} | Base64 인코딩된 OCSP 요청으로 인증서 상태를 조회 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp | DER 형식 OCSP 요청으로 인증서 상태를 조회 |

## 사전 준비하기

### 권한 관리

Private CA API는 역할 기반 접근 제어(RBAC)를 사용하며, 다음과 같이 구분됩니다.

- **VIEWER**: 저장소/인증서/템플릿 조회, 인증서 다운로드, CRL 조회 등 읽기 작업만 수행할 수 있습니다.
- **ADMIN**: 저장소/인증서/템플릿 생성·수정·삭제, CRL 수동 갱신 등 모든 관리 작업을 수행할 수 있습니다.
- **공개 엔드포인트**: CRL 다운로드(DER/PEM)와 OCSP API는 인증서 검증용으로 인증 없이 접근 가능합니다.

### 인증서 형식

Private CA API에서 사용하는 주요 인증서 형식은 다음과 같습니다.

- **PEM(privacy enhanced mail)**: 텍스트 기반 인증서 형식으로, Base64로 인코딩되어 있으며 `-----BEGIN CERTIFICATE-----`로 시작합니다. 사람이 읽을 수 있고 편집이 쉽습니다.
- **DER(distinguished encoding rules)**: 바이너리 형식 인증서로, PEM보다 파일 크기가 작고 효율적입니다. 주로 Java 애플리케이션에서 사용됩니다.

## 저장소 API

### 저장소 목록 조회

저장소 목록을 조회합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/ca-stores
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |

**Query Parameters**

| 이름 | 타입 | 필수 | 설명 | 제약 조건 |
|------|------|------|------|-----------|
| page | Number | N | 페이지 번호 | 0부터 시작<br>기본값: `0` |
| size | Number | N | 페이지 크기 | 기본값: `10` |
| search | String | N | 검색어 | 저장소 이름 또는 설명 |

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
    "caInfoList": [
      {
        "id": 1,
        "name": "Production CA Store",
        "description": "Production environment CA storage",
        "toastProjectId": 12345,
        "templateCnt": 3,
        "issuerCnt": 2,
        "certificateCnt": 25,
        "totalEabCnt": 0,
        "activeEabCnt": 0,
        "deletedEabCnt": 0,
        "status": "ACTIVE",
        "crlActive": true,
        "crlRefreshPeriod": 7,
        "ocspActive": true,
        "ocspRefreshPeriod": 1,
        "ocspUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp",
        "creationDatetime": "2024-01-15T10:00:00",
        "creationUser": "admin@example.com",
        "lastChangeDatetime": "2024-01-20T14:30:00",
        "lastChangeUser": "admin@example.com"
      }
    ],
    "totalCnt": 1,
    "totalPageNo": 1,
    "currentPageNo": 0
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| caInfoList | Array | 저장소 정보 목록 |
| caInfoList[].id | Long | 저장소 ID |
| caInfoList[].name | String | 저장소 이름 |
| caInfoList[].description | String | 저장소 설명 |
| caInfoList[].toastProjectId | Long | NHN Cloud 프로젝트 ID |
| caInfoList[].templateCnt | Long | 템플릿 개수 |
| caInfoList[].issuerCnt | Long | 발급자 인증서 개수 |
| caInfoList[].certificateCnt | Long | 전체 인증서 개수 |
| caInfoList[].status | String | 저장소 상태(`ACTIVE`, `INACTIVE`, `DELETED`) |
| caInfoList[].crlActive | Boolean | CRL 활성화 여부 |
| caInfoList[].crlRefreshPeriod | Number | CRL 갱신 주기(일) |
| caInfoList[].ocspActive | Boolean | OCSP 활성화 여부 |
| caInfoList[].ocspRefreshPeriod | Number | OCSP 갱신 주기(시간) |
| caInfoList[].ocspUrl | String | OCSP 서버 URL |
| totalCnt | Long | 전체 저장소 개수 |
| totalPageNo | Long | 전체 페이지 수 |
| currentPageNo | Long | 현재 페이지 번호(0부터 시작) |

### 저장소 상세 조회

저장소 상세 정보를 조회합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |

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
    "id": 1,
    "name": "Production CA Store",
    "description": "Production environment CA storage",
    "toastProjectId": 12345,
    "templateCnt": 3,
    "issuerCnt": 2,
    "certificateCnt": 25,
    "totalEabCnt": 0,
    "activeEabCnt": 0,
    "deletedEabCnt": 0,
    "status": "ACTIVE",
    "crlActive": true,
    "crlRefreshPeriod": 7,
    "ocspActive": true,
    "ocspRefreshPeriod": 1,
    "ocspUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp",
    "creationDatetime": "2024-01-15T10:00:00",
    "creationUser": "admin@example.com",
    "lastChangeDatetime": "2024-01-20T14:30:00",
    "lastChangeUser": "admin@example.com"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 저장소 ID |
| name | String | 저장소 이름 |
| description | String | 저장소 설명 |
| toastProjectId | Long | NHN Cloud 프로젝트 ID |
| templateCnt | Long | 템플릿 개수 |
| issuerCnt | Long | 발급자 인증서 개수 |
| certificateCnt | Long | 전체 인증서 개수 |
| status | String | 저장소 상태(`ACTIVE`, `INACTIVE`, `DELETED`) |
| crlActive | Boolean | CRL 활성화 여부 |
| crlRefreshPeriod | Number | CRL 갱신 주기(일) |
| ocspActive | Boolean | OCSP 활성화 여부 |
| ocspRefreshPeriod | Number | OCSP 갱신 주기(시간) |
| ocspUrl | String | OCSP 서버 URL |
| creationDatetime | LocalDateTime | 생성 일시 |
| creationUser | String | 생성자 |
| lastChangeDatetime | LocalDateTime | 마지막 변경 일시 |
| lastChangeUser | String | 마지막 변경자 |

### 저장소 생성

저장소를 생성합니다.

#### 요청

```
POST /v2.0/appkeys/{appkey}/ca-stores
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |

**Request Body**

| 이름 | 타입 | 필수 | 설명 | 제약 조건 |
|------|------|------|------|-----------|
| name | String | Y | 저장소 이름 | 최대 64자 |
| description | String | N | 저장소 설명 | 최대 256자 |
| crlActive | Boolean | N | CRL 활성화 여부 | 기본값: `false` |
| crlRefreshPeriod | Number | N | CRL 갱신 주기(일) | 1 ~ 30<br>기본값: `7` |
| ocspActive | Boolean | N | OCSP 활성화 여부 | 기본값: `false` |
| ocspRefreshPeriod | Number | N | OCSP 갱신 주기(시간) | 1 ~ 12<br>기본값: `1` |

**필요 권한**

- `ADMIN`

**요청 예시**

```json
{
  "name": "Production CA Store",
  "description": "Production environment CA storage",
  "crlActive": true,
  "crlRefreshPeriod": 7,
  "ocspActive": true,
  "ocspRefreshPeriod": 1
}
```

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
    "id": 1,
    "name": "Production CA Store",
    "toastProjectId": 12345,
    "status": "ACTIVE",
    "creationDatetime": "2024-01-15T10:00:00",
    "creationUser": "admin@example.com",
    "lastChangeDatetime": "2024-01-15T10:00:00",
    "lastChangeUser": "admin@example.com"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 생성된 저장소 ID |
| name | String | 저장소 이름 |
| toastProjectId | Long | NHN Cloud 프로젝트 ID |
| status | String | 저장소 상태 |
| creationDatetime | LocalDateTime | 생성 일시 |
| creationUser | String | 생성자 |
| lastChangeDatetime | LocalDateTime | 마지막 변경 일시 |
| lastChangeUser | String | 마지막 변경자 |

### 저장소 수정

저장소를 수정합니다.

#### 요청

```
PUT /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |

**Request Body**

저장소 생성과 동일합니다.

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
  "body": {
    "id": 1,
    "name": "Production CA Store (Updated)",
    "toastProjectId": 12345,
    "status": "ACTIVE"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 저장소 ID |
| name | String | 저장소 이름 |
| toastProjectId | Long | NHN Cloud 프로젝트 ID |
| status | String | 저장소 상태 |

### 저장소 삭제

저장소를 삭제합니다.

#### 요청

```
DELETE /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |

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
  "body": {
    "id": 1,
    "name": "Production CA Store",
    "toastProjectId": 12345,
    "status": "DELETED"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 삭제된 저장소 ID |
| name | String | 저장소 이름 |
| toastProjectId | Long | NHN Cloud 프로젝트 ID |
| status | String | 저장소 상태(`DELETED`) |

## 인증서(발급자) API

### 인증서 목록 조회

저장소에 포함된 인증서 목록을 조회합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |

**Query Parameters**

| 이름 | 타입 | 필수 | 설명 | 제약 조건 |
|------|------|------|------|-----------|
| type | String | N | 인증서 타입 필터 | `all`, `issuer`, `leaf`<br>기본값: `all` |
| page | Number | N | 페이지 번호 | 0부터 시작<br>기본값: `0` |
| size | Number | N | 페이지 크기 | 기본값: `10` |
| search | String | N | 검색어 | Common Name 기준 |

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
    "listCerts": [
      {
        "id": 10,
        "name": "Root CA",
        "type": "ROOT",
        "commonName": "NHN Cloud Root CA",
        "serialNumber": "12:34:56:78:90:AB:CD:EF",
        "status": "ACTIVE",
        "notBefore": "2024-01-15T10:00:00",
        "notAfter": "2034-01-15T10:00:00",
        "isLeaf": false,
        "creationDatetime": "2024-01-15T10:00:00",
        "creationUser": "admin@example.com"
      },
      {
        "id": 11,
        "name": "Intermediate CA",
        "type": "INTERMEDIATE",
        "commonName": "NHN Cloud Intermediate CA",
        "serialNumber": "AB:CD:EF:01:23:45:67:89",
        "status": "ACTIVE",
        "notBefore": "2024-01-15T10:30:00",
        "notAfter": "2029-01-15T10:30:00",
        "isLeaf": false,
        "creationDatetime": "2024-01-15T10:30:00",
        "creationUser": "admin@example.com"
      }
    ],
    "totalCnt": 2,
    "totalPageNo": 1,
    "currentPageNo": 0
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| listCerts | Array | 인증서 정보 목록 |
| listCerts[].id | Long | 인증서 ID |
| listCerts[].name | String | 인증서 이름 |
| listCerts[].type | String | 인증서 타입(`ROOT`, `INTERMEDIATE`, `LEAF`) |
| listCerts[].commonName | String | Common Name |
| listCerts[].serialNumber | String | 시리얼 번호(16진수, 콜론 구분) |
| listCerts[].status | String | 인증서 상태(`ACTIVE`, `INACTIVE`, `REVOKED`, `EXPIRED`, `DELETED`) |
| listCerts[].notBefore | LocalDateTime | 유효 시작일 |
| listCerts[].notAfter | LocalDateTime | 유효 만료일 |
| listCerts[].isLeaf | Boolean | Leaf 인증서 여부 |
| listCerts[].creationDatetime | LocalDateTime | 생성 일시 |
| listCerts[].creationUser | String | 생성자 |
| totalCnt | Long | 전체 개수 |
| totalPageNo | Long | 전체 페이지 수 |
| currentPageNo | Long | 현재 페이지 번호 |

### 인증서 상세 조회

인증서 상세 정보를 조회합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
| certId | Long | Y | 인증서 ID |

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
    "certificateId": 10,
    "name": "Root CA",
    "description": "Production Root CA",
    "type": "ROOT",
    "status": "ACTIVE",
    "commonName": "NHN Cloud Root CA",
    "serialNumber": "12:34:56:78:90:AB:CD:EF",
    "subjectInfo": {
      "country": "KR",
      "organization": "NHN Cloud",
      "commonName": "NHN Cloud Root CA"
    },
    "notAfterDateTime": "2034-01-15 10:00:00",
    "notBeforeDateTime": "2024-01-15 10:00:00",
    "keyAlgorithm": "RSA",
    "keyLength": 2048,
    "keyUsage": ["KEY_CERT_SIGN", "CRL_SIGN"],
    "extendedKeyUsage": [],
    "certificatePem": "-----BEGIN CERTIFICATE-----\nMIIDazCC...\n-----END CERTIFICATE-----",
    "chainCertificatePem": "-----BEGIN CERTIFICATE-----\nMIIDazCC...\n-----END CERTIFICATE-----",
    "signatureAlgorithm": "SHA256_WITH_RSA",
    "isLeaf": false,
    "childCertificateList": [
      {
        "certificateId": 11,
        "name": "Intermediate CA",
        "description": "Production Intermediate CA",
        "cn": "NHN Cloud Intermediate CA",
        "serialNumber": "AB:CD:EF:01:23:45:67:89",
        "isLeaf": false
      }
    ],
    "crlUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/certs/10/crl/der",
    "ocspUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp",
    "policies": []
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| certificateId | Long | 인증서 ID |
| name | String | 인증서 이름 |
| description | String | 인증서 설명 |
| type | String | 인증서 타입(`ROOT`, `INTERMEDIATE`, `LEAF`) |
| status | String | 인증서 상태 |
| commonName | String | Common Name |
| serialNumber | String | 시리얼 번호 |
| subjectInfo | Object | 주체 정보 |
| notAfterDateTime | LocalDateTime | 유효 만료일 |
| notBeforeDateTime | LocalDateTime | 유효 시작일 |
| keyAlgorithm | String | 키 알고리즘(`RSA`, `EC`, `ED25519`) |
| keyLength | Number | 키 길이 |
| keyUsage | String[] | Key Usage 목록 |
| extendedKeyUsage | String[] | Extended Key Usage 목록 |
| certificatePem | String | 인증서 PEM |
| chainCertificatePem | String | 인증서 체인 PEM |
| signatureAlgorithm | String | 서명 알고리즘 |
| isLeaf | Boolean | Leaf 인증서 여부 |
| childCertificateList | Array | 하위 인증서 목록 |
| crlUrl | String | CRL 다운로드 URL |
| ocspUrl | String | OCSP 서버 URL |
| policies | String[] | Certificate Policies OID 목록 |

### 인증서(발급자) 발급

ROOT 또는 INTERMEDIATE 인증서(발급자)를 발급합니다.

#### 요청

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |

!!! danger "주의"
    ROOT, INTERMEDIATE 인증서만 발급 가능합니다. LEAF 인증서는 템플릿으로 발급해야 합니다.

**Request Body**

| 이름 | 타입 | 필수 | 설명 | 제약 조건 |
|------|------|------|------|-----------|
| certificateType | String | Y | 인증서 유형 | `ROOT` 또는 `INTERMEDIATE` |
| parentCertId | Number | 조건부 | 상위 인증서 ID | `certificateType`이 `INTERMEDIATE`일 경우 필수 |
| name | String | Y | 인증서 이름 | 최대 64자 |
| description | String | N | 인증서 설명 | 최대 256자 |
| subjectInfo | Object | Y | 주체 정보 | `commonName` 필수 |
| ttlValue | Number | 조건부 | 유효 기간(초) | 0 ~ 315,360,000(최대 10년)<br>`specificDate`와 택 1 |
| specificDate | String | 조건부 | 만료 일시 | 1970-01-01T00:00:00 ~ 2999-12-31T23:59:59<br>형식: `2025-12-31T23:59:59`<br>`ttlValue`와 택 1 |
| backDateValidation | Number | N | 백데이트 유효성(초) | 0 ~ 2,592,000(최대 30일)<br>기본값: `30` |
| maxDepth | Number | N | 하위 CA 생성 가능 깊이 | -1 ~ 3<br>기본값: `0` |
| keyInfo | Object | Y | 키 정보 | 하단 참조 |
| signatureAlgorithm | String | Y | 서명 알고리즘 | 하단 참조<br>선택한 Key에 맞는 서명 알고리즘 필수(보통 SHA256 형태 선택) |
| excludeCommonNameFromSans | Boolean | N | CN을 SAN에서 제외 | 기본값: `false` |
| sans | String[] | N | DNS SAN 목록 | |
| ipSans | String[] | N | IP SAN 목록 | |
| urlSans | String[] | N | URL SAN 목록 | |
| otherSans | OidInfo[] | N | 기타 SAN 목록 | 하단 OidInfo 참조 |

**KeyInfo**

| algorithm | keySize |
|-----------|---------|
| RSA | 2048, 3072, 4096 |
| EC 또는 ECDSA | 224, 256, 384, 521 |
| ED25519 | 256(고정, 값을 입력해도 무시됨) |

**SignatureAlgorithm**

| 알고리즘 | 호환 키 |
|----------|---------|
| SHA256_WITH_RSA | RSA |
| SHA384_WITH_RSA | RSA |
| SHA512_WITH_RSA | RSA |
| SHA256_WITH_ECDSA | EC 또는 ECDSA |
| SHA384_WITH_ECDSA | EC 또는 ECDSA |
| SHA512_WITH_ECDSA | EC 또는 ECDSA |
| ED25519 | ED25519 |

**SubjectInfo**

| 이름 | 타입 | 설명 | 제약 조건 |
|------|------|------|-----------|
| commonName | String | Common Name(CN) | 최대 64자 |
| serialNumber | String | 시리얼 번호 | 최대 64자 |
| country | String | 국가 코드(C) | ISO 3166, 2자리 |
| stateOrProvince | String | 주/도(ST) | 최대 128자 |
| locality | String | 도시/지역(L) | 최대 128자 |
| streetAddress | String | 거리 주소(STREET) | 최대 128자 |
| postalCode | String | 우편번호 | 최대 40자 |
| organization | String | 조직명(O) | 최대 64자 |
| organizationalUnit | String | 부서명(OU) | 최대 64자 |

**OidInfo(기타 SAN)**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| oid | String | Y | OID(예: `1.2.840.113549.1.9.1`) |
| type | String | Y | `UTF8String`, `IA5String`, `PrintableString`, `BMPString`, `UniversalString` |
| value | String | Y | 값(최대 255자) |

**필요 권한**

- `ADMIN`

**요청 예시**

```json
{
  "name": "Root CA",
  "description": "Production Root CA",
  "certificateType": "ROOT",
  "keyInfo": {
    "algorithm": "RSA",
    "keySize": 2048
  },
  "subjectInfo": {
    "commonName": "NHN Cloud Root CA",
    "organization": "NHN Cloud",
    "country": "KR"
  },
  "ttlValue": 315360000,
  "signatureAlgorithm": "SHA256_WITH_RSA",
  "maxDepth": 2
}
```

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
    "certificateId": 10,
    "name": "Root CA",
    "description": "Production Root CA",
    "type": "ROOT",
    "status": "ACTIVE",
    "commonName": "NHN Cloud Root CA",
    "serialNumber": "12:34:56:78:90:AB:CD:EF",
    "subjectInfo": {
      "country": "KR",
      "organization": "NHN Cloud",
      "commonName": "NHN Cloud Root CA"
    },
    "notAfterDateTime": "2034-01-15 10:00:00",
    "notBeforeDateTime": "2024-01-15 10:00:00",
    "keyAlgorithm": "RSA",
    "keyLength": 2048,
    "keyUsage": ["KEY_CERT_SIGN", "CRL_SIGN"],
    "certificatePem": "-----BEGIN CERTIFICATE-----\nMIIDazCC...\n-----END CERTIFICATE-----",
    "chainCertificatePem": "-----BEGIN CERTIFICATE-----\nMIIDazCC...\n-----END CERTIFICATE-----",
    "signatureAlgorithm": "SHA256_WITH_RSA",
    "isLeaf": false,
    "crlUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/certs/10/crl/der",
    "ocspUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| certificateId | Long | 발급된 인증서 ID |
| name | String | 인증서 이름 |
| type | String | 인증서 타입(`ROOT`, `INTERMEDIATE`) |
| status | String | 인증서 상태 |
| commonName | String | Common Name |
| serialNumber | String | 시리얼 번호 |
| notAfterDateTime | LocalDateTime | 유효 만료일 |
| notBeforeDateTime | LocalDateTime | 유효 시작일 |
| keyAlgorithm | String | 키 알고리즘 |
| keyLength | Number | 키 길이 |
| certificatePem | String | 발급된 인증서 PEM |
| chainCertificatePem | String | 인증서 체인 PEM |
| signatureAlgorithm | String | 서명 알고리즘 |
| crlUrl | String | CRL 다운로드 URL |
| ocspUrl | String | OCSP 서버 URL |

!!! note "참고"
    발급자 인증서 발급 API는 `privateKey`를 응답에 포함하지 않습니다. 발급자의 개인 키는 서버에 안전하게 저장됩니다.

### 인증서 폐기

인증서를 폐기합니다.

#### 요청

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/revoke
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
| certId | Long | Y | 인증서 ID |

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
  "body": {
    "result": true,
    "serialNumber": "12:34:56:78:90:AB:CD:EF",
    "revocationDatetime": "2024-06-01T15:30:00"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| result | Boolean | 폐기 처리 성공 여부 |
| serialNumber | String | 폐기된 인증서의 시리얼 번호 |
| revocationDatetime | LocalDateTime | 폐기 일시 |

## 템플릿 API

### 템플릿 목록 조회

템플릿 목록을 조회합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |

**Query Parameters**

| 이름 | 타입 | 필수 | 설명 | 제약 조건 |
|------|------|------|------|-----------|
| page | Number | N | 페이지 번호 | 0부터 시작<br>기본값: `0` |
| size | Number | N | 페이지 크기 | 기본값: `10` |
| search | String | N | 검색어 | 템플릿 이름 |

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
    "templates": [
      {
        "id": 100,
        "name": "Web Server Template",
        "description": "Server certificate template for web servers"
      },
      {
        "id": 101,
        "name": "Client Auth Template",
        "description": "Client authentication certificate template"
      }
    ],
    "totalCnt": 2,
    "totalPageNo": 1,
    "currentPageNo": 0
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| templates | Array | 템플릿 정보 목록 |
| templates[].id | Long | 템플릿 ID |
| templates[].name | String | 템플릿 이름 |
| templates[].description | String | 템플릿 설명 |
| totalCnt | Long | 전체 템플릿 개수 |
| totalPageNo | Number | 전체 페이지 수 |
| currentPageNo | Number | 현재 페이지 번호(0부터 시작) |

### 템플릿 상세 조회

템플릿 상세 정보를 조회합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
| templateId | Long | Y | 템플릿 ID |

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
    "id": 100,
    "name": "Web Server Template",
    "description": "Server certificate template for web servers",
    "subjectInfo": {
      "country": "KR",
      "organization": "NHN Cloud"
    },
    "keyAlgorithm": "RSA",
    "keyLength": 2048,
    "keyInfo": {
      "algorithm": "RSA",
      "keySize": 2048
    },
    "keyUsage": ["DIGITAL_SIGNATURE", "KEY_ENCIPHERMENT"],
    "extendedKeyUsage": ["SERVER_AUTHENTICATION"],
    "extendedKeyUsageOids": [],
    "signatureAlgorithm": "SHA256_WITH_RSA",
    "isLeaf": true,
    "signingCertificateId": 11,
    "signingCertificateName": "Intermediate CA",
    "creationUser": "admin@example.com",
    "creationDatetime": "2024-01-15 11:00:00",
    "lastChangeUser": "admin@example.com",
    "lastChangeDatetime": "2024-01-15 11:00:00",
    "basicConstraintsValidForNonCa": false,
    "storeInServer": true,
    "allowIpSans": false,
    "useCsrCommonName": false,
    "useCsrSans": false,
    "useCsrOtherFields": false,
    "maxTTL": "31536000",
    "signatureBits": 256,
    "backDateValidation": 30,
    "urlSansWhitelist": [],
    "otherSansWhitelist": [],
    "policies": [],
    "otherFields": []
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 템플릿 ID |
| name | String | 템플릿 이름 |
| description | String | 템플릿 설명 |
| subjectInfo | Object | 주체 정보(템플릿 고정 값) |
| keyAlgorithm | String | 키 알고리즘 |
| keyLength | Number | 키 길이 |
| keyInfo | Object | 키 정보 |
| keyUsage | String[] | Key Usage 목록 |
| extendedKeyUsage | String[] | Extended Key Usage 목록 |
| extendedKeyUsageOids | String[] | Extended Key Usage 커스텀 OID 목록 |
| signatureAlgorithm | String | 서명 알고리즘 |
| signingCertificateId | Long | 서명에 사용되는 발급자 인증서 ID |
| signingCertificateName | String | 서명에 사용되는 발급자 인증서 이름 |
| basicConstraintsValidForNonCa | Boolean | Non-CA에 대한 Basic Constraints 검사 여부 |
| storeInServer | Boolean | 서버 저장 여부 |
| allowIpSans | Boolean | IP SAN 허용 여부 |
| useCsrCommonName | Boolean | CSR의 CN 사용 여부 |
| useCsrSans | Boolean | CSR의 SAN 사용 여부 |
| useCsrOtherFields | Boolean | CSR의 기타 필드 사용 여부 |
| maxTTL | String | 최대 TTL(초 단위 문자열) |
| signatureBits | Number | 서명 비트 수 |
| backDateValidation | Long | 백데이트 검증 값(초) |
| urlSansWhitelist | String[] | URL SAN 화이트리스트 |
| otherSansWhitelist | OidInfo[] | 기타 SAN 화이트리스트 |
| policies | String[] | 정책 OID 목록 |
| otherFields | OidInfo[] | 기타 필드 |
| creationDatetime | LocalDateTime | 생성 일시 |
| creationUser | String | 생성자 |
| lastChangeDatetime | LocalDateTime | 마지막 변경 일시 |
| lastChangeUser | String | 마지막 변경자 |

### 템플릿 생성

템플릿을 생성합니다.

#### 요청

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |

**Request Body**

| 이름 | 타입 | 필수 | 설명 | 제약 조건 |
|------|------|------|------|-----------|
| name | String | Y | 템플릿 이름 | 최대 64자 |
| description | String | N | 템플릿 설명 | 최대 256자 |
| parentCertId | Number | Y | 상위 인증서 ID | 발급자 인증서만 가능 |
| maxTTL | Number | 조건부 | 최대 유효 기간(초) | 0 ~ 315,360,000(최대 10년)<br>`maxSpecificDate`와 택 1 |
| maxSpecificDate | String | 조건부 | 최대 만료일 제한 | 1970-01-01T00:00:00 ~ 2999-12-31T23:59:59<br>형식: `2025-12-31T23:59:59`<br>`maxTTL`과 택 1 |
| backDateValidation | Number | N | 백데이트 유효성(초) | 0 ~ 2,592,000(최대 30일)<br>기본값: `30` |
| allowIpSans | Boolean | N | IP SAN 허용 여부 | 기본값: `false` |
| urlSansWhitelist | String[] | N | URL SAN 화이트리스트 | |
| otherSansWhitelist | OidInfo[] | N | 기타 SAN 화이트리스트 | |
| storeInServer | Boolean | N | 서버에 인증서 저장 여부 | 기본값: `true` |
| basicConstraintsValidForNonCa | Boolean | N | Non-CA에 대한 Basic Constraints 검사 | 기본값: `false` |
| useCsrCommonName | Boolean | N | CSR의 CN 사용 여부 | 기본값: `false` |
| useCsrSans | Boolean | N | CSR의 SAN 사용 여부 | 기본값: `false` |
| keyInfo | Object | Y | 키 정보 | 하단 참조 |
| signatureBits | Number | N | 서명 비트 수 | `256`, `384`, `512`<br>기본값: `256`<br>ED25519의 경우 무시됨 |
| keyUsage | String[] | N | 키 사용 용도 | 하단 참조 |
| extendedKeyUsage | String[] | N | 확장 키 사용 용도 | 하단 참조 |
| extendedKeyUsageOids | String[] | N | 확장 키 사용 커스텀 OID | |
| policies | String[] | N | 정책 OID 목록 | |
| subjectInfo | Object | N | 주체 정보 | 하단 참조 |
| useCsrOtherFields | Boolean | N | CSR의 기타 필드 사용 여부 | 기본값: `false` |
| otherFields | OidInfo[] | N | 기타 필드 | 하단 참조 |

**KeyInfo**

| algorithm | keySize |
|-----------|---------|
| RSA | 2048, 3072, 4096 |
| EC 또는 ECDSA | 224, 256, 384, 521 |
| ED25519 | 256(고정, 값을 입력해도 무시됨) |

**Key Usage 값**

- `DIGITAL_SIGNATURE`
- `NON_REPUDIATION`
- `KEY_ENCIPHERMENT`
- `DATA_ENCIPHERMENT`
- `KEY_AGREEMENT`
- `KEY_CERT_SIGN`
- `CRL_SIGN`
- `ENCIPHER_ONLY`
- `DECIPHER_ONLY`

**Extended Key Usage 값**

- `SERVER_AUTHENTICATION`
- `CLIENT_AUTHENTICATION`
- `CODE_SIGNING`
- `EMAIL_PROTECTION`
- `TIME_STAMPING`
- `OCSP_SIGNING`
- `ANY_EXTENDED_KEY_USAGE`

**SubjectInfo**

| 이름 | 타입 | 설명 | 제약 조건 |
|------|------|------|-----------|
| serialNumber | String | 시리얼 번호 | 최대 64자 |
| country | String | 국가 코드(C) | ISO 3166, 2자리 |
| stateOrProvince | String | 주/도(ST) | 최대 128자 |
| locality | String | 도시/지역(L) | 최대 128자 |
| streetAddress | String | 거리 주소(STREET) | 최대 128자 |
| postalCode | String | 우편번호 | 최대 40자 |
| organization | String | 조직명(O) | 최대 64자 |
| organizationalUnit | String | 부서명(OU) | 최대 64자 |

**OidInfo(기타 필드)**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| oid | String | Y | OID(예: `1.2.840.113549.1.9.1`) |
| type | String | Y | `UTF8String`, `IA5String`, `PrintableString`, `BMPString`, `UniversalString` |
| value | String | Y | 값(최대 255자) |

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
  "body": {
    "templateId": 100,
    "name": "Web Server Template",
    "description": "Server certificate template for web servers",
    "signingCertificateId": 11,
    "signingCertificateName": "Intermediate CA"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| templateId | Long | 생성된 템플릿 ID |
| name | String | 템플릿 이름 |
| description | String | 템플릿 설명 |
| signingCertificateId | Long | 서명에 사용되는 발급자 인증서 ID |
| signingCertificateName | String | 서명에 사용되는 발급자 인증서 이름 |

### 템플릿 수정

템플릿을 수정합니다.

#### 요청

```
PUT /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
| templateId | Long | Y | 템플릿 ID |

**Request Body**

템플릿 생성과 동일합니다.

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

| 필드 | 타입 | 설명 |
|------|------|------|
| body | Boolean | 수정 성공 여부 |

### 템플릿 삭제

템플릿을 삭제합니다.

#### 요청

```
DELETE /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
| templateId | Long | Y | 템플릿 ID |

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

| 필드 | 타입 | 설명 |
|------|------|------|
| body | Boolean | 삭제 성공 여부 |

### 템플릿으로 인증서 발급

템플릿을 사용하여 인증서를 발급합니다.

#### 요청

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}/certificates
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
| templateId | Long | Y | 템플릿 ID |

**Request Body**

| 이름 | 타입 | 필수 | 설명 | 제약 조건 |
|------|------|------|------|-----------|
| mode | String | Y | 인증서 발급 모드 | `GENERATE` 또는 `SIGN` |
| commonName | String | Y | Common Name | 최대 64자 |
| ttlValue | Number | 조건부 | 유효 기간(초) | 0 ~ 315,360,000(최대 10년)<br>`specificDate`와 택 1 |
| specificDate | String | 조건부 | 특정 만료 날짜 | 1970-01-01T00:00:00 ~ 2999-12-31T23:59:59<br>형식: `2025-12-31T23:59:59`<br>`ttlValue`와 택 1 |
| csr | String | 조건부 | CSR | SIGN 모드 시 필수 |
| format | String | N | 인증서 형식 | PEM, DER, PEM_BUNDLE<br>기본값: `PEM`<br>잘못된 값 입력 시 기본값으로 적용 |
| privateKeyFormat | String | N | 개인 키 형식 | PEM, DER, PKCS8<br>기본값: `PEM`<br>잘못된 값 입력 시 기본값으로 적용<br>GENERATE 모드 시에만 사용 |
| removeRootsFromChain | Boolean | N | 체인에서 루트 제거 | SIGN 모드 시에만 사용 |
| excludeCommonNameFromSans | Boolean | N | CN을 SAN에서 제외 | |
| serialNumber | String | N | Serial Number | 최대 64자 |
| sans | String[] | N | DNS SAN 목록 | |
| ipSans | String[] | N | IP SAN 목록 | |
| urlSans | String[] | N | URL SAN 목록 | |
| otherSans | OidInfo[] | N | 기타 SAN 목록 | 하단 참조 |

**OidInfo(기타 SAN)**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| oid | String | Y | OID(예: `1.2.840.113549.1.9.1`) |
| type | String | Y | `UTF8String`, `IA5String`, `PrintableString`, `BMPString`, `UniversalString` |
| value | String | Y | 값(최대 255자) |

**필요 권한**

- `ADMIN`

**요청 예시(GENERATE 모드)**

```json
{
  "mode": "GENERATE",
  "commonName": "api.example.com",
  "ttlValue": 31536000,
  "sans": ["api.example.com", "www.example.com"],
  "format": "PEM",
  "privateKeyFormat": "PKCS8"
}
```

**요청 예시(SIGN 모드)**

```json
{
  "mode": "SIGN",
  "csr": "-----BEGIN CERTIFICATE REQUEST-----\nMIIC...\n-----END CERTIFICATE REQUEST-----",
  "ttlValue": 31536000,
  "format": "PEM",
  "removeRootsFromChain": false
}
```

#### 응답

**Response Body**(GENERATE 모드)

```json
{
  "header": {
    "resultCode": 0,
    "resultMessage": "SUCCESS",
    "isSuccessful": true
  },
  "body": {
    "certificateId": 200,
    "name": "api.example.com",
    "type": "LEAF",
    "status": "ACTIVE",
    "commonName": "api.example.com",
    "serialNumber": "AB:CD:EF:11:22:33:44:55",
    "subjectInfo": {
      "country": "KR",
      "organization": "NHN Cloud",
      "commonName": "api.example.com"
    },
    "notAfterDateTime": "2025-06-01 15:30:00",
    "notBeforeDateTime": "2024-06-01 15:30:00",
    "keyAlgorithm": "RSA",
    "keyLength": 2048,
    "keyUsage": ["DIGITAL_SIGNATURE", "KEY_ENCIPHERMENT"],
    "extendedKeyUsage": ["SERVER_AUTHENTICATION"],
    "certificatePem": "-----BEGIN CERTIFICATE-----\nMIIDazCC...\n-----END CERTIFICATE-----",
    "chainCertificatePem": "-----BEGIN CERTIFICATE-----\nMIIDazCC...\n-----END CERTIFICATE-----",
    "signatureAlgorithm": "SHA256_WITH_RSA",
    "isLeaf": true,
    "crlUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/certs/11/crl/der",
    "ocspUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp",
    "privateKey": "-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBgkqhkiG9w0...\n-----END PRIVATE KEY-----"
  }
}
```

**Response Body**(SIGN 모드)

```json
{
  "header": {
    "resultCode": 0,
    "resultMessage": "SUCCESS",
    "isSuccessful": true
  },
  "body": {
    "certificateId": 201,
    "name": "api.example.com",
    "type": "LEAF",
    "status": "ACTIVE",
    "commonName": "api.example.com",
    "serialNumber": "AB:CD:EF:11:22:33:44:66",
    "subjectInfo": {
      "country": "KR",
      "organization": "NHN Cloud",
      "commonName": "api.example.com"
    },
    "notAfterDateTime": "2025-06-01 15:30:00",
    "notBeforeDateTime": "2024-06-01 15:30:00",
    "keyAlgorithm": "RSA",
    "keyLength": 2048,
    "keyUsage": ["DIGITAL_SIGNATURE", "KEY_ENCIPHERMENT"],
    "extendedKeyUsage": ["SERVER_AUTHENTICATION"],
    "certificatePem": "-----BEGIN CERTIFICATE-----\nMIIDazCC...\n-----END CERTIFICATE-----",
    "chainCertificatePem": "-----BEGIN CERTIFICATE-----\nMIIDazCC...\n-----END CERTIFICATE-----",
    "signatureAlgorithm": "SHA256_WITH_RSA",
    "isLeaf": true,
    "crlUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/certs/11/crl/der",
    "ocspUrl": "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp"
  }
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| certificateId | Long | 발급된 인증서 ID |
| name | String | 인증서 이름 |
| type | String | 인증서 타입(`LEAF`) |
| status | String | 인증서 상태 |
| commonName | String | Common Name |
| serialNumber | String | 시리얼 번호 |
| notAfterDateTime | LocalDateTime | 유효 만료일 |
| notBeforeDateTime | LocalDateTime | 유효 시작일 |
| keyAlgorithm | String | 키 알고리즘 |
| keyLength | Number | 키 길이 |
| certificatePem | String | 발급된 인증서 PEM |
| chainCertificatePem | String | 인증서 체인 PEM |
| signatureAlgorithm | String | 서명 알고리즘 |
| crlUrl | String | CRL 다운로드 URL |
| ocspUrl | String | OCSP 서버 URL |
| privateKey | String | 개인 키(GENERATE 모드일 때만 반환) |

!!! danger "주의"
    GENERATE 모드로 발급 시 응답에 포함된 `privateKey`는 **이 응답이 유일한 반환 시점**입니다. 서버에는 저장되지 않으므로 즉시 안전한 위치에 저장하세요. SIGN 모드는 클라이언트가 개인 키를 보유하므로 응답에 `privateKey`가 포함되지 않습니다.

## 인증서 다운로드 API

### 인증서 다운로드

발급된 인증서를 PEM 형식으로 다운로드합니다.

#### 요청

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/download
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
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
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
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
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/der
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
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
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/pem
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
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
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
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
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp/{ocspRequestBase64}
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |
| ocspRequestBase64 | String | Y | Base64로 인코딩된 OCSP 요청 |

!!! danger "주의"
    OCSP 요청을 Base64로 인코딩할 때는 URL-safe 형태로 변환해야 합니다.

    - 표준 Base64 인코딩 후 다음 문자들을 변환합니다.
        - `+` → `-`(plus를 hyphen으로)
        - `/` → `_`(slash를 underscore로)
    - Padding 문자(`=`)는 제거합니다.
    - 예시
        - 변환 전: `MEow/SDAwL+oGCC+sGAQUF/BzAh==`
        - 변환 후: `MEow_SDAwL-oGCC-sGAQUF_BzAh`(`+` → `-`, `/` → `_`, `=` 제거)

**필요 권한**

- 권한 체크 없음(공개 엔드포인트)

**요청 예시**

```sh
# OCSP 요청 생성 및 Base64 인코딩
OCSP_REQUEST=$(openssl ocsp -issuer ca.pem -cert cert.pem -reqout - | base64 -w 0)

# URL 인코딩된 요청 전송
curl -X GET "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp/${OCSP_REQUEST}"
```

#### 응답

**Response Headers**

- Content-Type: `application/ocsp-response`

**Response Body**

OCSP 응답(DER 형식)

### OCSP 상태 조회(POST)

DER 형식 OCSP 요청을 처리합니다.

#### 요청

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp
```

**Path Parameters**

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| appkey | String | Y | 앱키 |
| caStoreId | Long | Y | 저장소 ID |

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
curl -X POST "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp" \
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
