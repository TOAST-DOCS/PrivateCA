# API v2.0 Guide
**Management > Private CA > API v2.0 Guide**

You can use the NHN Cloud Private CA API to manage certificates programmatically.

## Basic Info

### API Endpoint

| Region | Endpoint |
| --- | --- |
| KR1 | https://kr1-pca.api.nhncloudservice.com |

### API List

| Method | URI | Description |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{certId}/download | Download a certificate in PEM format. |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl | Retrieve CRL information (PEM, issue/renew time). |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/der | Download a CRL in DER format. |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/pem | Download a CRL in PEM format. |
| POST | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl | Manually renew a CRL. |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/ocsp/{ocspRequestBase64} | Retrieve certificate status with a Base64-encoded OCSP request. |
| POST | /v2.0/appkeys/{appkey}/cas/{caId}/ocsp | Retrieve certificate status with an OCSP request in DER format. |

## Prepare in advance

### Authentication

All API requests require the following authentication headers:

```
X-NHN-Authorization: Bearer {access_token}
```

!!! TIP "Notice"
- You can read more about the authentication token required in the authorization header [here](https://docs.nhncloud.com/en/nhncloud/en/public-api/api-authentication/).
- The appkey can be found in the console and must be included in all API paths.

### Manage permissions

The Private CA API uses role-based access control (RBAC), which is categorized as follows:

- **VIEWER**: You can only perform read operations, such as downloading certificates and looking up CRLs.
- **ADMIN**: You can perform all administrative tasks, including manually renewing CRLs.
- **Public endpoints**: The CRL download (DER/PEM) and OCSP APIs are accessible without authentication for certificate validation.

### Certificate formats

The main certificate formats used by the Private CA API are as follows:

- **Privacy enhanced mail (PEM)**: A text-based certificate format, encoded in Base64 and starting ` with -----BEGIN CERTIFICATE-----`. Human-readable and easy to edit.
- **Distinguished encoding rules (DER)**: A certificate in binary format, which is smaller and more efficient than PEM. It is primarily used in Java applications.

## Certificate download API

### Download the certificate

Download the issued certificate in PEM format.

#### Request

```
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{certId}/download
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caId | Long | Y | Repository ID |
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
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caId | Long | Y | Repository ID |
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
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/der
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caId | Long | Y | Repository ID |
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
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/pem
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caId | Long | Y | Repository ID |
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
POST /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caId | Long | Y | Repository ID |
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
GET /v2.0/appkeys/{appkey}/cas/{caId}/ocsp/{ocspRequestBase64}
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App Key |
| caId | Long | Y | Store ID |
| ocspRequestBase64 | String | Y | Base64-encoded OCSP request |

!!! danger "Caution"
When encoding OCSP requests to Base64, they must be converted to a URL-safe form.

    - Convert the following characters after standard Base64 encoding:
        - `+` → `-` (plus as hyphen)
        - `/` → `_` (slash to underscore)
    - Remove the padding character (`=`).
    - Example:
        - Before conversion: `MEow/SDAwL+oGCC+sGAQUF/BzAh==`
        - After conversion: `MEow_SDAwL-oGCC-sGAQUF_BzAh` (`+` → `-`, `/` → `_`, `=` removed)

**Required Permission**

- No permission checks (public endpoints)

**Request Example**

```sh
# OCSP request creation and Base64 encoding
OCSP_REQUEST=$(openssl ocsp -issuer ca.pem -cert cert.pem -reqout - | base64 -w 0)

# Send URL-encoded requests
curl -X GET "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/cas/1/ocsp/${OCSP_REQUEST}"
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
POST /v2.0/appkeys/{appkey}/cas/{caId}/ocsp
```

**Path Parameters**

| Name | Type | Required | Description |
|------|------|------|------|
| appkey | String | Y | App Key |
| caId | Long | Y | Repository ID |

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
curl -X POST "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/cas/1/ocsp" \
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
