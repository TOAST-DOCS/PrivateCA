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

#### Store

| Method | URI | Description |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores | Retrieves a list of stores |
| POST | /v2.0/appkeys/{appkey}/ca-stores | Creates a store |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | Retrieves detailed information of a store |
| PUT | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | Modifies a store |
| DELETE | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | Deletes a store |

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

- **VIEWER**: You can only perform read operations, such as viewing stores/certificates/templates, downloading certificates and looking up CRLs.
- **ADMIN**: You can perform all administrative tasks, including creating·modifying·deleting stores/certificates/templates, manually renewing CRLs.
- **Public endpoints**: The CRL download (DER/PEM) and OCSP APIs are accessible without authentication for certificate validation.

### Certificate formats

The main certificate formats used by the Private CA API are as follows:

- **Privacy enhanced mail (PEM)**: A text-based certificate format, encoded in Base64 and starting ` with -----BEGIN CERTIFICATE-----`. Human-readable and easy to edit.
- **Distinguished encoding rules (DER)**: A certificate in binary format, which is smaller and more efficient than PEM. It is primarily used in Java applications.


## Store API

### List Stores

Retrieves a list of stores.

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
| search | String | N | Search keyword | Store name or description |

**Required Permissions**

- `VIEWER` or higher

### Get Store Details

Retrieves detailed information of a store.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Store ID |

**Required Permissions**

- `VIEWER` or higher

### Create Store

Creates a store.

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
| name | String | Y | Store name | Maximum 64 characters |
| description | String | N | Store description | Maximum 256 characters |
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

### Modify Store

Modifies a store.

#### Request

```
PUT /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Store ID |

**Request Body**

Same as Create Store.

**Required Permissions**

- `ADMIN`

### Delete Store

Deletes a store.

#### Request

```
DELETE /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Store ID |

**Required Permissions**

- `ADMIN`

## Certificate (Issuer) API

### List Certificates

Retrieves a list of certificates included in a store.

#### Request

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App key |
| caStoreId | Long | Y | Store ID |

**Query Parameters**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| type | String | N | Certificate type filter | `all`, `issuer`, `leaf`<br>Default: `all` |
| page | Number | N | Page number | Starts from 0<br>Default: `0` |
| size | Number | N | Page size | Default: `10` |
| search | String | N | Search keyword | Based on Common Name |

**Required Permissions**

- `VIEWER` or higher

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
| caStoreId | Long | Y | Store ID |
| certId | Long | Y | Certificate ID |

**Required Permissions**

- `VIEWER` or higher

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
| caStoreId | Long | Y | Store ID |

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
| caStoreId | Long | Y | Store ID |
| certId | Long | Y | Certificate ID |

**Required Permissions**

- `ADMIN`

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
| caStoreId | Long | Y | Store ID |

**Query Parameters**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| page | Number | N | Page number | Starts from 0<br>Default: `0` |
| size | Number | N | Page size | Default: `10` |
| search | String | N | Search keyword | Template name |

**Required Permissions**

- `VIEWER` or higher

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
| caStoreId | Long | Y | Store ID |
| templateId | Long | Y | Template ID |

**Required Permissions**

- `VIEWER` or higher

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
| caStoreId | Long | Y | Store ID |

**Request Body**

| Name | Type | Required | Description | Constraints |
|------|------|------|------|-----------|
| name | String | Y | Template name | Maximum 64 characters |
| description | String | N | Template description | Maximum 256 characters |
| parentCertId | Number | Y | Parent certificate ID | |
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
| caStoreId | Long | Y | Store ID |
| templateId | Long | Y | Template ID |

**Request Body**

Same as Create Template.

**Required Permissions**

- `ADMIN`

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
| caStoreId | Long | Y | Store ID |
| templateId | Long | Y | Template ID |

**Required Permissions**

- `ADMIN`

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
| caStoreId | Long | Y | Store ID |
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
| caStoreId | Long | Y | Store ID |
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
3. Check and adjust the CRL refresh`period (crlRefreshPeriod`).

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
