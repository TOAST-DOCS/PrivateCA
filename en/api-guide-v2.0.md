## API v2.0 Guide
**Management > Private CA > API v2.0 Guide**

You can use the NHN Cloud Private CA API to manage certificates programmatically.

## Private CA API Common Information

### API Endpoint

| Region | Endpoint |
| --- | --- |
| KR1 | https://kr1-pca.api.nhncloudservice.com |

### Authentication and Authorization

Private CA API v2.0 supports Appkey and User Access Key token as authentication methods for API calls.

Appkey is included in the request URL when calling the API to identify and point to a specific resource.
A User Access Key token is a temporary, Bearer-type access token issued from a User Access Key, used for authentication and authorization when calling the API.

For more information on how to check and use each authentication method, see [Appkey](/nhncloud/en/public-api/appkey/) and [User Access Key Token](/nhncloud/en/public-api/user-access-key-token/).

### API List

#### Repository

| Method | URI | Description |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores | Retrieves a list of repositories |
| POST | /v2.0/appkeys/{appkey}/ca-stores | Creates a repository |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | Retrieves detailed information of a repository |
| PUT | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | Modifies a repository |
| DELETE | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | Deletes a repository |

#### Certificate (Issuer)

| Method | URI | Description |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs | Retrieves a list of certificates |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs | Issues a certificate (issuer) (ROOT and INTERMEDIATE only) |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId} | Retrieves detailed information of a certificate |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/revoke | Revokes a certificate |

#### Template

| Method | URI | Description |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates | Retrieves a list of templates |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates | Creates a template |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId} | Retrieves detailed information of a template |
| PUT | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId} | Modifies a template |
| DELETE | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId} | Deletes a template |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}/certificates | Issues a certificate using a template |

#### Certificate Download

| Method | URI | Description |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/download | Downloads a certificate in PEM format |

#### CRL

| Method | URI | Description |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl | Retrieves CRL information (PEM, issuance/renewal time) |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/der | Downloads a CRL in DER format |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/pem | Downloads a CRL in PEM format |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl | Manually renews a CRL |

#### OCSP

| Method | URI | Description |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp/{ocspRequestBase64} | Retrieves certificate status using a Base64-encoded OCSP request |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp | Retrieves certificate status using a DER format OCSP request |

## Prepare in advance

### Manage permissions

The Private CA API uses role-based access control (RBAC), which is categorized as follows:

- **VIEWER**: You can only perform read operations, such as viewing repositories/certificates/templates, downloading certificates and looking up CRLs.
- **ADMIN**: You can perform all administrative tasks, including creating·modifying·deleting repositories/certificates/templates, manually renewing CRLs.
- **Public endpoints**: The CRL download (DER/PEM) and OCSP APIs are accessible without authentication for certificate validation.

### Certificate formats

The main certificate formats used by the Private CA API are as follows:

- **Privacy enhanced mail (PEM)**: A text-based certificate format, encoded in Base64 and starting with `-----BEGIN CERTIFICATE-----`. Human-readable and easy to edit.
- **Distinguished encoding rules (DER)**: A certificate in binary format, which is smaller and more efficient than PEM. It is primarily used in Java applications.

## Repository API

### List Repositories

Retrieves a list of repositories.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |

**Query Parameters**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| page | Number | N | Page number | Starts from 0<br>Default: `0` |
| size | Number | N | Page size | Default: `10` |
| search | String | N | Search keyword | Repository name or description |

**Required Permissions**

- `VIEWER` or higher

#### Response

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

| Field | Type | Description |
|------|------|------|
| caInfoList | Array | List of repository information |
| caInfoList[].id | Long | Repository ID |
| caInfoList[].name | String | Repository name |
| caInfoList[].description | String | Repository description |
| caInfoList[].toastProjectId | Long | NHN Cloud project ID |
| caInfoList[].templateCnt | Long | Number of templates |
| caInfoList[].issuerCnt | Long | Number of issuer certificates |
| caInfoList[].certificateCnt | Long | Total number of certificates |
| caInfoList[].status | String | Repository status (`ACTIVE`, `INACTIVE`, `DELETED`) |
| caInfoList[].crlActive | Boolean | Whether CRL is enabled |
| caInfoList[].crlRefreshPeriod | Number | CRL renewal cycle (days) |
| caInfoList[].ocspActive | Boolean | Whether OCSP is enabled |
| caInfoList[].ocspRefreshPeriod | Number | OCSP renewal cycle (hours) |
| caInfoList[].ocspUrl | String | OCSP server URL |
| totalCnt | Long | Total number of repositories |
| totalPageNo | Long | Total number of pages |
| currentPageNo | Long | Current page number (starts from 0) |

### Get Repository Details

Retrieves detailed information of a repository.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |

**Required Permissions**

- `VIEWER` or higher

#### Response

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

| Field | Type | Description |
|------|------|------|
| id | Long | Repository ID |
| name | String | Repository name |
| description | String | Repository description |
| toastProjectId | Long | NHN Cloud project ID |
| templateCnt | Long | Number of templates |
| issuerCnt | Long | Number of issuer certificates |
| certificateCnt | Long | Total number of certificates |
| status | String | Repository status (`ACTIVE`, `INACTIVE`, `DELETED`) |
| crlActive | Boolean | Whether CRL is enabled |
| crlRefreshPeriod | Number | CRL renewal cycle (days) |
| ocspActive | Boolean | Whether OCSP is enabled |
| ocspRefreshPeriod | Number | OCSP renewal cycle (hours) |
| ocspUrl | String | OCSP server URL |
| creationDatetime | LocalDateTime | Creation date |
| creationUser | String | Creator |
| lastChangeDatetime | LocalDateTime | Last modified date |
| lastChangeUser | String | Last modified by |

### Create Repository

Creates a repository.

#### Request

```
POST /v2.0/appkeys/{appkey}/ca-stores
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |

**Request Body**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| name | String | Y | Repository name | Maximum 64 characters |
| description | String | N | Repository description | Maximum 256 characters |
| crlActive | Boolean | N | CRL enabled | Default: `false` |
| crlRefreshPeriod | Number | N | CRL renewal cycle (days) | 1 ~ 30<br>Default: `7` |
| ocspActive | Boolean | N | OCSP enabled | Default: `false` |
| ocspRefreshPeriod | Number | N | OCSP renewal cycle (hours) | 1 ~ 12<br>Default: `1` |

**Required Permissions**

- `ADMIN`

**Request Example**

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

#### Response

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

| Field | Type | Description |
|------|------|------|
| id | Long | Created repository ID |
| name | String | Repository name |
| toastProjectId | Long | NHN Cloud project ID |
| status | String | Repository status |
| creationDatetime | LocalDateTime | Creation date |
| creationUser | String | Creator |
| lastChangeDatetime | LocalDateTime | Last modified date |
| lastChangeUser | String | Last modified by |

### Modify Repository

Modifies a repository.

#### Request

```
PUT /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |

**Request Body**

Same as Create Repository.

**Required Permissions**

- `ADMIN`

#### Response

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

| Field | Type | Description |
|------|------|------|
| id | Long | Repository ID |
| name | String | Repository name |
| toastProjectId | Long | NHN Cloud project ID |
| status | String | Repository status |

### Delete Repository

Deletes a repository.

#### Request

```
DELETE /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |

**Required Permissions**

- `ADMIN`

#### Response

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

| Field | Type | Description |
|------|------|------|
| id | Long | Deleted repository ID |
| name | String | Repository name |
| toastProjectId | Long | NHN Cloud project ID |
| status | String | Repository status (`DELETED`) |

## Certificate (Issuer) API

### List Certificates

Retrieves a list of certificates included in a repository.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |

**Query Parameters**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| type | String | N | Certificate type filter | `all`, `issuer`, `leaf`<br>Default: `all` |
| page | Number | N | Page number | Starts from 0<br>Default: `0` |
| size | Number | N | Page size | Default: `10` |
| search | String | N | Search keyword | Based on Common Name |

**Required Permissions**

- `VIEWER` or higher

#### Response

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

| Field | Type | Description |
|------|------|------|
| listCerts | Array | List of certificate information |
| listCerts[].id | Long | Certificate ID |
| listCerts[].name | String | Certificate name |
| listCerts[].type | String | Certificate type (`ROOT`, `INTERMEDIATE`, `LEAF`) |
| listCerts[].commonName | String | Common Name |
| listCerts[].serialNumber | String | Serial number (hexadecimal, colon-separated) |
| listCerts[].status | String | Certificate status (`ACTIVE`, `INACTIVE`, `REVOKED`, `EXPIRED`, `DELETED`) |
| listCerts[].notBefore | LocalDateTime | Validity start date |
| listCerts[].notAfter | LocalDateTime | Validity end date |
| listCerts[].isLeaf | Boolean | Whether it is a Leaf certificate |
| listCerts[].creationDatetime | LocalDateTime | Creation date |
| listCerts[].creationUser | String | Creator |
| totalCnt | Long | Total count |
| totalPageNo | Long | Total number of pages |
| currentPageNo | Long | Current page number |

### Get Certificate Details

Retrieves detailed information of a certificate.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |
| certId | Long | Y | Certificate ID |

**Required Permissions**

- `VIEWER` or higher

#### Response

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

| Field | Type | Description |
|------|------|------|
| certificateId | Long | Certificate ID |
| name | String | Certificate name |
| description | String | Certificate description |
| type | String | Certificate type (`ROOT`, `INTERMEDIATE`, `LEAF`) |
| status | String | Certificate status |
| commonName | String | Common Name |
| serialNumber | String | Serial number |
| subjectInfo | Object | Subject information |
| notAfterDateTime | LocalDateTime | Validity end date |
| notBeforeDateTime | LocalDateTime | Validity start date |
| keyAlgorithm | String | Key algorithm (`RSA`, `EC`, `ED25519`) |
| keyLength | Number | Key length |
| keyUsage | String[] | Key Usage list |
| extendedKeyUsage | String[] | Extended Key Usage list |
| certificatePem | String | Certificate PEM |
| chainCertificatePem | String | Certificate chain PEM |
| signatureAlgorithm | String | Signature algorithm |
| isLeaf | Boolean | Whether it is a Leaf certificate |
| childCertificateList | Array | List of child certificates |
| crlUrl | String | CRL download URL |
| ocspUrl | String | OCSP server URL |
| policies | String[] | Certificate Policies OID list |

### Issue Certificate (Issuer)

Issues a ROOT or INTERMEDIATE certificate (issuer).

#### Request

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |

!!! danger "Caution"
    Only ROOT and INTERMEDIATE certificates can be issued. LEAF certificates must be issued using a template.

**Request Body**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| certificateType | String | Y | Certificate type | `ROOT` or `INTERMEDIATE` |
| parentCertId | Number | Conditional | Parent certificate ID | Required if `certificateType` is `INTERMEDIATE` |
| name | String | Y | Certificate name | Maximum 64 characters |
| description | String | N | Certificate description | Maximum 256 characters |
| subjectInfo | Object | Y | Subject information | `commonName` required |
| ttlValue | Number | Conditional | Validity period (seconds) | 0 ~ 315,360,000 (max 10 years)<br>Either `ttlValue` or `specificDate` |
| specificDate | String | Conditional | Expiration date and time | 1970-01-01T00:00:00 ~ 2999-12-31T23:59:59<br>Format: `2025-12-31T23:59:59`<br>Either `specificDate` or `ttlValue` |
| backDateValidation | Number | N | Back-date validation (seconds) | 0 ~ 2,592,000 (max 30 days)<br>Default: `30` |
| maxDepth | Number | N | Maximum depth for sub-CA creation | -1 ~ 3<br>Default: `0` |
| keyInfo | Object | Y | Key information | See below |
| signatureAlgorithm | String | Y | Signature algorithm | See below<br>Must match the selected key (SHA256 format recommended) |
| excludeCommonNameFromSans | Boolean | N | Exclude CN from SAN | Default: `false` |
| sans | String[] | N | DNS SAN list | |
| ipSans | String[] | N | IP SAN list | |
| urlSans | String[] | N | URL SAN list | |
| otherSans | OidInfo[] | N | Other SAN list | See OidInfo below |

**KeyInfo**

| algorithm | keySize |
|-----------|---------|
| RSA | 2048, 3072, 4096 |
| EC or ECDSA | 224, 256, 384, 521 |
| ED25519 | 256 (fixed, value is ignored even if entered) |

**SignatureAlgorithm**

| Algorithm | Compatible Key |
|----------|---------|
| SHA256_WITH_RSA | RSA |
| SHA384_WITH_RSA | RSA |
| SHA512_WITH_RSA | RSA |
| SHA256_WITH_ECDSA | EC or ECDSA |
| SHA384_WITH_ECDSA | EC or ECDSA |
| SHA512_WITH_ECDSA | EC or ECDSA |
| ED25519 | ED25519 |

**SubjectInfo**

| Name | Type | Description | Constraints |
|------|------|------|-----------|
| commonName | String | Common Name (CN) | Maximum 64 characters |
| serialNumber | String | Serial number | Maximum 64 characters |
| country | String | Country code (C) | ISO 3166, 2 characters |
| stateOrProvince | String | State/province (ST) | Maximum 128 characters |
| locality | String | City/region (L) | Maximum 128 characters |
| streetAddress | String | Street address (STREET) | Maximum 128 characters |
| postalCode | String | Postal code | Maximum 40 characters |
| organization | String | Organization name (O) | Maximum 64 characters |
| organizationalUnit | String | Department name (OU) | Maximum 64 characters |

**OidInfo (Other SAN)**

| Name | Type | Required | Description |
|------|------|------|------|
| oid | String | Y | OID (e.g., `1.2.840.113549.1.9.1`) |
| type | String | Y | `UTF8String`, `IA5String`, `PrintableString`, `BMPString`, `UniversalString` |
| value | String | Y | Value (maximum 255 characters) |

**Required Permissions**

- `ADMIN`

**Request Example**

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

#### Response

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

| Field | Type | Description |
|------|------|------|
| certificateId | Long | Issued certificate ID |
| name | String | Certificate name |
| type | String | Certificate type (`ROOT`, `INTERMEDIATE`) |
| status | String | Certificate status |
| commonName | String | Common Name |
| serialNumber | String | Serial number |
| notAfterDateTime | LocalDateTime | Validity end date |
| notBeforeDateTime | LocalDateTime | Validity start date |
| keyAlgorithm | String | Key algorithm |
| keyLength | Number | Key length |
| certificatePem | String | Issued certificate PEM |
| chainCertificatePem | String | Certificate chain PEM |
| signatureAlgorithm | String | Signature algorithm |
| crlUrl | String | CRL download URL |
| ocspUrl | String | OCSP server URL |

!!! note "Note"
    The issuer certificate issuance API does not include `privateKey` in the response. The issuer's private key is stored securely on the server.

### Revoke Certificate

Revokes a certificate.

#### Request

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/revoke
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |
| certId | Long | Y | Certificate ID |

**Required Permissions**

- `ADMIN`

#### Response

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

| Field | Type | Description |
|------|------|------|
| result | Boolean | Whether the revocation was successful |
| serialNumber | String | Serial number of the revoked certificate |
| revocationDatetime | LocalDateTime | Revocation date |

## Template API

### List Templates

Retrieves a list of templates.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |

**Query Parameters**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| page | Number | N | Page number | Starts from 0<br>Default: `0` |
| size | Number | N | Page size | Default: `10` |
| search | String | N | Search keyword | Template name |

**Required Permissions**

- `VIEWER` or higher

#### Response

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

| Field | Type | Description |
|------|------|------|
| templates | Array | List of template information |
| templates[].id | Long | Template ID |
| templates[].name | String | Template name |
| templates[].description | String | Template description |
| totalCnt | Long | Total number of templates |
| totalPageNo | Number | Total number of pages |
| currentPageNo | Number | Current page number (starts from 0) |

### Get Template Details

Retrieves detailed information of a template.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |
| templateId | Long | Y | Template ID |

**Required Permissions**

- `VIEWER` or higher

#### Response

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

| Field | Type | Description |
|------|------|------|
| id | Long | Template ID |
| name | String | Template name |
| description | String | Template description |
| subjectInfo | Object | Subject information (template fixed value) |
| keyAlgorithm | String | Key algorithm |
| keyLength | Number | Key length |
| keyInfo | Object | Key information |
| keyUsage | String[] | Key Usage list |
| extendedKeyUsage | String[] | Extended Key Usage list |
| extendedKeyUsageOids | String[] | Extended Key Usage custom OID list |
| signatureAlgorithm | String | Signature algorithm |
| signingCertificateId | Long | ID of the issuer certificate used for signing |
| signingCertificateName | String | Name of the issuer certificate used for signing |
| basicConstraintsValidForNonCa | Boolean | Whether Basic Constraints validation applies for non-CA |
| storeInServer | Boolean | Whether to store on the server |
| allowIpSans | Boolean | Whether IP SAN is allowed |
| useCsrCommonName | Boolean | Whether to use the CN of the CSR |
| useCsrSans | Boolean | Whether to use the SAN of the CSR |
| useCsrOtherFields | Boolean | Whether to use other fields of the CSR |
| maxTTL | String | Maximum TTL (string in seconds) |
| signatureBits | Number | Signature bit length |
| backDateValidation | Long | Back-date validation value (seconds) |
| urlSansWhitelist | String[] | URL SAN whitelist |
| otherSansWhitelist | OidInfo[] | Other SAN whitelist |
| policies | String[] | Policy OID list |
| otherFields | OidInfo[] | Other fields |
| creationDatetime | LocalDateTime | Creation date |
| creationUser | String | Creator |
| lastChangeDatetime | LocalDateTime | Last modified date |
| lastChangeUser | String | Last modified by |

### Create Template

Creates a template.

#### Request

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |

**Request Body**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| name | String | Y | Template name | Maximum 64 characters |
| description | String | N | Template description | Maximum 256 characters |
| parentCertId | Number | Y | Parent certificate ID | Only issuer certificates allowed |
| maxTTL | Number | Conditional | Maximum validity period (seconds) | 0 ~ 315,360,000 (max 10 years)<br>Either `maxTTL` or `maxSpecificDate` |
| maxSpecificDate | String | Conditional | Maximum expiration date limit | 1970-01-01T00:00:00 ~ 2999-12-31T23:59:59<br>Format: `2025-12-31T23:59:59`<br>Either `maxSpecificDate` or `maxTTL` |
| backDateValidation | Number | N | Back-date validation (seconds) | 0 ~ 2,592,000 (max 30 days)<br>Default: `30` |
| allowIpSans | Boolean | N | IP SAN enabled | Default: `false` |
| urlSansWhitelist | String[] | N | URL SAN whitelist | |
| otherSansWhitelist | OidInfo[] | N | Other SAN whitelist | |
| storeInServer | Boolean | N | Certificate storage on server enabled | Default: `true` |
| basicConstraintsValidForNonCa | Boolean | N | Basic Constraints validation for non-CA | Default: `false` |
| useCsrCommonName | Boolean | N | CSR CN enabled | Default: `false` |
| useCsrSans | Boolean | N | CSR SAN enabled | Default: `false` |
| keyInfo | Object | Y | Key information | See below |
| signatureBits | Number | N | Signature bit length | `256`, `384`, `512`<br>Default: `256`<br>Ignored for ED25519 |
| keyUsage | String[] | N | Key usage | See below |
| extendedKeyUsage | String[] | N | Extended key usage | See below |
| extendedKeyUsageOids | String[] | N | Extended key usage custom OID | |
| policies | String[] | N | Policy OID list | |
| subjectInfo | Object | N | Subject information | See below |
| useCsrOtherFields | Boolean | N | CSR other fields enabled | Default: `false` |
| otherFields | OidInfo[] | N | Other fields | See below |

**KeyInfo**

| algorithm | keySize |
|-----------|---------|
| RSA | 2048, 3072, 4096 |
| EC or ECDSA | 224, 256, 384, 521 |
| ED25519 | 256 (fixed, value is ignored even if entered) |

**Key Usage value**

- `DIGITAL_SIGNATURE`
- `NON_REPUDIATION`
- `KEY_ENCIPHERMENT`
- `DATA_ENCIPHERMENT`
- `KEY_AGREEMENT`
- `KEY_CERT_SIGN`
- `CRL_SIGN`
- `ENCIPHER_ONLY`
- `DECIPHER_ONLY`

**Extended Key Usage value**

- `SERVER_AUTHENTICATION`
- `CLIENT_AUTHENTICATION`
- `CODE_SIGNING`
- `EMAIL_PROTECTION`
- `TIME_STAMPING`
- `OCSP_SIGNING`
- `ANY_EXTENDED_KEY_USAGE`

**SubjectInfo**

| Name | Type | Description | Constraints |
|------|------|------|-----------|
| serialNumber | String | Serial number | Maximum 64 characters |
| country | String | Country code (C) | ISO 3166, 2 characters |
| stateOrProvince | String | State/province (ST) | Maximum 128 characters |
| locality | String | City/region (L) | Maximum 128 characters |
| streetAddress | String | Street address (STREET) | Maximum 128 characters |
| postalCode | String | Postal code | Maximum 40 characters |
| organization | String | Organization name (O) | Maximum 64 characters |
| organizationalUnit | String | Department name (OU) | Maximum 64 characters |

**OidInfo (Other Fields)**

| Name | Type | Required | Description |
|------|------|------|------|
| oid | String | Y | OID (e.g., `1.2.840.113549.1.9.1`) |
| type | String | Y | `UTF8String`, `IA5String`, `PrintableString`, `BMPString`, `UniversalString` |
| value | String | Y | Value (maximum 255 characters) |

**Required Permissions**

- `ADMIN`

#### Response

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

| Field | Type | Description |
|------|------|------|
| templateId | Long | Created template ID |
| name | String | Template name |
| description | String | Template description |
| signingCertificateId | Long | ID of the issuer certificate used for signing |
| signingCertificateName | String | Name of the issuer certificate used for signing |

### Modify Template

Modifies a template.

#### Request

```
PUT /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |
| templateId | Long | Y | Template ID |

**Request Body**

Same as Create Template.

**Required Permissions**

- `ADMIN`

#### Response

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

| Field | Type | Description |
|------|------|------|
| body | Boolean | Whether the modification was successful |

### Delete Template

Deletes a template.

#### Request

```
DELETE /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |
| templateId | Long | Y | Template ID |

**Required Permissions**

- `ADMIN`

#### Response

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

| Field | Type | Description |
|------|------|------|
| body | Boolean | Whether the deletion was successful |

### Issue Certificate Using Template

Issues a certificate using a template.

#### Request

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}/certificates
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Repository ID |
| templateId | Long | Y | Template ID |

**Request Body**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| mode | String | Y | Certificate issuance mode | `GENERATE` or `SIGN` |
| commonName | String | Y | Common Name | Maximum 64 characters |
| ttlValue | Number | Conditional | Validity period (seconds) | 0 ~ 315,360,000 (max 10 years)<br>Either `ttlValue` or `specificDate` |
| specificDate | String | Conditional | Specific expiration date | 1970-01-01T00:00:00 ~ 2999-12-31T23:59:59<br>Format: `2025-12-31T23:59:59`<br>Either `specificDate` or `ttlValue` |
| csr | String | Conditional | CSR | Required for SIGN mode |
| format | String | N | Certificate format | PEM, DER, PEM_BUNDLE<br>Default: `PEM`<br>Invalid values will use default |
| privateKeyFormat | String | N | Private key format | PEM, DER, PKCS8<br>Default: `PEM`<br>Invalid values will use default<br>Available in GENERATE mode only |
| removeRootsFromChain | Boolean | N | Remove root from chain | Available in SIGN mode only |
| excludeCommonNameFromSans | Boolean | N | Exclude CN from SAN | |
| serialNumber | String | N | Serial Number | Maximum 64 characters |
| sans | String[] | N | DNS SAN list | |
| ipSans | String[] | N | IP SAN list | |
| urlSans | String[] | N | URL SAN list | |
| otherSans | OidInfo[] | N | Other SAN list | See below |

**OidInfo (Other SAN)**

| Name | Type | Required | Description |
|------|------|------|------|
| oid | String | Y | OID (e.g., `1.2.840.113549.1.9.1`) |
| type | String | Y | `UTF8String`, `IA5String`, `PrintableString`, `BMPString`, `UniversalString` |
| value | String | Y | Value (maximum 255 characters) |

**Required Permissions**

- `ADMIN`

**Request Example (GENERATE mode)**

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

**Request Example (SIGN mode)**

```json
{
  "mode": "SIGN",
  "csr": "-----BEGIN CERTIFICATE REQUEST-----\nMIIC...\n-----END CERTIFICATE REQUEST-----",
  "ttlValue": 31536000,
  "format": "PEM",
  "removeRootsFromChain": false
}
```

#### Response

**Response Body** (GENERATE mode)

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

**Response Body** (SIGN mode)

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

| Field | Type | Description |
|------|------|------|
| certificateId | Long | Issued certificate ID |
| name | String | Certificate name |
| type | String | Certificate type (`LEAF`) |
| status | String | Certificate status |
| commonName | String | Common Name |
| serialNumber | String | Serial number |
| notAfterDateTime | LocalDateTime | Validity end date |
| notBeforeDateTime | LocalDateTime | Validity start date |
| keyAlgorithm | String | Key algorithm |
| keyLength | Number | Key length |
| certificatePem | String | Issued certificate PEM |
| chainCertificatePem | String | Certificate chain PEM |
| signatureAlgorithm | String | Signature algorithm |
| crlUrl | String | CRL download URL |
| ocspUrl | String | OCSP server URL |
| privateKey | String | Private key (returned only in GENERATE mode) |

!!! danger "Caution"
    When issuing in GENERATE mode, the `privateKey` included in the response is **the only time it is returned**. It is not stored on the server, so save it immediately to a secure location. In SIGN mode, the client holds the private key, so `privateKey` is not included in the response.

## Certificate download API

### Download the certificate

Download the issued certificate in PEM format.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/download
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | Repository ID |
| certId | Long | Y | Certificate ID |

**Required Permission**

- `VIEWER` and above

#### Response

**Response Headers**

- Content-Type: `application/x-pem-file`
- Content-Disposition: `attachment; filename={filename}.pem`

**Response Body**

Certificate data (PEM format)

## CRL API

A certificate revocation list (CRL) is a mechanism that provides a list of certificates issued by a particular issuer that have been revoked. Clients can download the CRL to verify that the certificate has been revoked.

### Retrieve CRL information

Retrieve CRL information for a specific issuer.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | Repository ID |
| issuerCertId | Long | Y | Issuer certificate ID |

**Required Permission**

- `VIEWER` and above

#### Response

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

| Field | Type | Description |
|------|------|------|
| crlPem | String | CRL PEM format data |
| thisUpdate | LocalDateTime | CRL issue time |
| nextUpdate | LocalDateTime | Next CRL expected time |

### Download the CRL (DER format)

Download the CRL in DER (binary) format.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/der
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | Repository ID |
| issuerCertId | Long | Y | Issuer certificate ID |

**Required Permission**

- No permission checks (public endpoints)

#### Response

**Response Headers**

- Content-Type: `application/pkix-crl`
- Content-Disposition: `attachment; filename=crl_{caId}_{certId}.crl`

**Response Body**

CRL data (DER format)

### Download the CRL (PEM format)

Download the CRL in PEM format.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/pem
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | Repository ID |
| issuerCertId | Long | Y | Issuer certificate ID |

**Required Permission**

- No permission checks (public endpoints)

#### Response

**Response Headers**

- Content-Type: `application/pkix-crl`
- Content-Disposition: `attachment; filename=crl_{caId}_{certId}.pem`

**Response Body**

CRL data (PEM format)

### Manually renew a CRL

Renew the CRL manually.

#### Request

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | Repository ID |
| issuerCertId | Long | Y | Issuer certificate ID |

**Required Permission**

- `ADMIN`

#### Response

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

The online certificate status protocol (OCSP) is a protocol that allows you to quickly check the revocation status of individual certificates. Unlike CRLs, you can look up the status of a specific certificate at the time of the request without downloading the entire list.

### Get OCSP Status (GET)

Processes Base64-encoded OCSP requests.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp/{ocspRequestBase64}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App Key |
| caStoreId | Long | Y | Repository ID |
| ocspRequestBase64 | String | Y | Base64-encoded OCSP request |

!!! danger "Caution"
    When encoding OCSP requests to Base64, they must be converted to a URL-safe form.

    - Convert the following characters after standard Base64 encoding:
        - `+` → `-` (plus as hyphen)
        - `/` → `_` (slash to underscore)
    - Remove the padding character (`=`).
    - Example
        - Before conversion: `MEow/SDAwL+oGCC+sGAQUF/BzAh==`
        - After conversion: `MEow_SDAwL-oGCC-sGAQUF_BzAh` (`+` → `-`, `/` → `_`, `=` removed)

**Required Permission**

- No permission checks (public endpoints)

**Request Example**

```sh
# OCSP request creation and Base64 encoding
OCSP_REQUEST=$(openssl ocsp -issuer ca.pem -cert cert.pem -reqout - | base64 -w 0)

# Send URL-encoded requests
curl -X GET "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp/${OCSP_REQUEST}"
```

#### Response

**Response Headers**

- Content-Type: `application/ocsp-response`

**Response Body**

OCSP response (DER format)

### OCSP Status Query (POST)

Process OCSP requests in DER format.

#### Request

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App Key |
| caStoreId | Long | Y | Repository ID |

**Required Permission**

- No permission checks (public endpoints)

**Request Headers**

- Content-Type: `application/ocsp-request`

**Request Body**

OCSP requests (DER format)

**Request Example**

```sh
# Create an OCSP request
openssl ocsp -issuer ca.pem -cert cert.pem -reqout ocsp-request.der

# Send OCSP requests
curl -X POST "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp" \
    -H "Content-Type: application/ocsp-request" \
    --data-binary @ocsp-request.der \
    -o ocsp-response.der

# Verify the OCSP response
openssl ocsp -respin ocsp-response.der -text
```

#### Response

**Response Headers**

- Content-Type: `application/ocsp-response`

**Response Body**

OCSP response (DER format)

## Troubleshooting

### If the CRL is not renewed

1. In the console, under Repository details, verify that CRLs are enabled.
2. Call the manual renewal API to renew immediately.
3. Check and adjust the CRL refresh period (`crlRefreshPeriod`).

### If there is no OCSP response

1. In the console, under Repository details, verify that OCSP is enabled.
2. Make sure you're using the correct repository ID.
3. Verify that the OCSP request is in the correct format (DER).

### OCSP response results differ from the actual certificate status

1. OCSP responses are cached based on the renewal cycle, so if you recently revoked a certificate, the old state might be returned until the renewal cycle has passed.
2. In the console, under Repository details, check the OCSP renewal cycle.
3. If you need to see the latest status immediately, look it up again after the renewal cycle has passed.

### If the certificate doesn't have a CRL/OCSP URL

- The extension was not included when the certificate was issued.
- After you enable CRL/OCSP in your repository settings, you must reissue the certificate.

!!! danger "Caution"
    - The CRL/OCSP URL is included when the certificate is issued, so the certificate must be reissued after changing the settings.
    - You can't change the CRL/OCSP URL of an already issued certificate.
