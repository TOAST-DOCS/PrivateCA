## API v2.0 가이드
**Management > Private CA > API v2.0 가이드**

NHN Cloud Private CA API를 사용하여 인증서를 프로그래밍 방식으로 관리할 수 있습니다.

## 기본 정보

### API 엔드포인트

| 리전 | 엔드포인트 |
| --- | --- |
| KR1 | https://kr1-pca.api.nhncloudservice.com |

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

### 인증

모든 API 요청에는 다음과 같은 인증 헤더가 필요합니다.

```
X-NHN-Authorization: Bearer {access_token}
```

!!! tip "알아두기"
    - 인증 헤더에 필요한 인증 토큰에 대한 자세한 사항은 [여기](https://docs.nhncloud.com/ko/nhncloud/ko/public-api/user-access-key-token/)에서 자세히 확인하실 수 있습니다.
    - 앱키(appkey)는 콘솔에서 확인할 수 있으며, 모든 API 경로에 포함되어야 합니다.

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

**필요 권한**

- `VIEWER` 이상

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
| crlRefreshPeriod | Number | N | CRL 갱신 주기(일) | 1~30<br>기본값: 7 |
| ocspActive | Boolean | N | OCSP 활성화 여부 | 기본값: `false` |
| ocspRefreshPeriod | Number | N | OCSP 갱신 주기(시간) | 1~12<br>기본값: 1 |

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

**필요 권한**

- `VIEWER` 이상

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
| ttlValue | Number | 조건부 | 유효 기간(초) | `specificDate`와 택 1, 0~315,360,000(최대 10년) |
| specificDate | String | 조건부 | 만료 일시 | `ttlValue`와 택 1, 형식: `2025-12-31T23:59:59`, `1970-01-01T00:00:00`~`2999-12-31T23:59:59` |
| backDateValidation | Number | N | 백데이트 유효성(초) | 0~2,592,000(최대 30일)<br>기본값: `30` |
| maxDepth | Number | N | 하위 CA 생성 가능 깊이 | -1~3<br>기본값: `0` |
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

**필요 권한**

- `VIEWER` 이상

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
| parentCertId | Number | Y | 상위 인증서 ID | |
| maxTTL | Number | 조건부 | 최대 유효 기간(초) | `maxSpecificDate`와 택 1, 0~315,360,000(최대 10년) |
| maxSpecificDate | String | 조건부 | 최대 만료일 제한 | `maxTTL`과 택 1, 형식: `2025-12-31T23:59:59`, `1970-01-01T00:00:00`~`2999-12-31T23:59:59` |
| backDateValidation | Number | N | 백데이트 유효성(초) | 0~2,592,000(최대 30일)<br>기본값: `30` |
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
| ttlValue | Number | 조건부 | 유효 기간(초) | `specificDate`와 택 1, 0~315,360,000(최대 10년) |
| specificDate | String | 조건부 | 특정 만료 날짜 | `ttlValue`와 택 1, 형식: `2025-12-31T23:59:59`, `1970-01-01T00:00:00`~`2999-12-31T23:59:59` |
| csr | String | 조건부 | CSR | SIGN 모드 시 필수 |
| format | String | N | 인증서 형식 | `PEM`, `DER`, `PEM_BUNDLE`<br>기본값: `PEM`<br>잘못된 값 입력 시 기본값으로 적용 |
| privateKeyFormat | String | N | 개인 키 형식 | `PEM`, `DER`, `PKCS8`<br>기본값: `PEM`<br>잘못된 값 입력 시 기본값으로 적용<br>GENERATE 모드 시에만 사용 |
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
