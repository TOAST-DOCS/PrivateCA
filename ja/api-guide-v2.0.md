## API v2.0ガイド
**Management > Private CA > API v2.0ガイド**

NHN Cloud Private CA APIを使用して証明書をプログラムで管理できます。

## Private CA API共通情報

### APIエンドポイント

| リージョン | エンドポイント |
| --- | --- |
| KR1 | https://kr1-pca.api.nhncloudservice.com |

### 認証および権限

Private CA API v2.0は、API呼び出しおよび認証のための認証方法として、Appkey、User Access Keyトークンをサポートしています。

Appkeyは、API呼び出し時にリクエストURLに含めて特定のリソースを指定・識別するために使用します。
User Access Keyトークンは、User Access Keyをもとに発行されるBearerタイプの一時的なアクセストークンであり、API呼び出し時の認証/認可に使用します。

各認証方法の確認手順や使用方法の詳細は、それぞれ[Appkey](/nhncloud/ja/public-api/appkey/)、[User Access Keyトークン](/nhncloud/ja/public-api/user-access-key-token/)をご参照ください。

### API一覧

#### リポジトリ

| Method | URI | 説明 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores | リポジトリ一覧を照会 |
| POST | /v2.0/appkeys/{appkey}/ca-stores | リポジトリを作成 |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | リポジトリ詳細情報を照会 |
| PUT | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | リポジトリを修正 |
| DELETE | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | リポジトリを削除 |

#### 証明書(発行者)

| Method | URI | 説明 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs | 証明書一覧を照会 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs | 証明書(発行者)を発行(ROOT、INTERMEDIATEのみ発行可能) |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId} | 証明書詳細情報を照会 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/revoke | 証明書を失効させる |

#### テンプレート

| Method | URI | 説明 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates | テンプレート一覧を照会 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates | テンプレートを作成 |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId} | テンプレート詳細情報を照会 |
| PUT | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId} | テンプレートを修正 |
| DELETE | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId} | テンプレートを削除 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}/certificates | テンプレートで証明書を発行 |

#### 証明書ダウンロード

| Method | URI | 説明 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/download | PEM形式の証明書をダウンロード |

#### CRL

| Method | URI | 説明 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl | CRL情報(PEM、発行/更新時間)を照会 |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/der | DER形式のCRLをダウンロード |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/pem | PEM形式のCRLをダウンロード |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl | CRLを手動で更新 |

#### OCSP

| Method | URI | 説明 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp/{ocspRequestBase64} | Base64エンコードされたOCSPリクエストで証明書の状態を照会 |
| POST | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp | DER形式のOCSPリクエストで証明書の状態を照会 |

## 事前準備

### 権限管理

Private CA APIはロールベースアクセス制御(RBAC)を使用し、次のように区分されます。

- **VIEWER**: リポジトリ/証明書/テンプレートの照会、証明書ダウンロード、CRL照会などの読み取り操作のみ実行できます。
- **ADMIN**: リポジトリ/証明書/テンプレートの作成・修正・削除、CRL手動更新など、すべての管理操作を実行できます。
- **パブリックエンドポイント**: CRLダウンロード(DER/PEM)とOCSP APIは、証明書検証用として認証なしでアクセス可能です。

### 証明書形式

Private CA APIで使用する主な証明書形式は次のとおりです。

- **PEM(privacy enhanced mail)**：テキストベースの証明書形式で、Base64でエンコードされており、`-----BEGIN CERTIFICATE-----`で始まります。人間が読むことができ、編集が容易です。
- **DER(distinguished encoding rules)**：バイナリ形式の証明書で、PEMよりファイルサイズが小さく効率的です。主にJavaアプリケーションで使用されます。

## リポジトリAPI

### リポジトリ一覧照会

リポジトリ一覧を照会します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |

**Query Parameters**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| page | Number | N | ページ番号 | 0から開始<br>デフォルト値: `0` |
| size | Number | N | ページサイズ | デフォルト値: `10` |
| search | String | N | 検索キーワード | リポジトリ名または説明 |

**必要権限**

- `VIEWER`以上

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| caInfoList | Array | リポジトリ情報一覧 |
| caInfoList[].id | Long | リポジトリID |
| caInfoList[].name | String | リポジトリ名 |
| caInfoList[].description | String | リポジトリ説明 |
| caInfoList[].toastProjectId | Long | NHN CloudプロジェクトID |
| caInfoList[].templateCnt | Long | テンプレート数 |
| caInfoList[].issuerCnt | Long | 発行者証明書数 |
| caInfoList[].certificateCnt | Long | 全証明書数 |
| caInfoList[].status | String | リポジトリ状態(`ACTIVE`, `INACTIVE`, `DELETED`) |
| caInfoList[].crlActive | Boolean | CRL有効化の有無 |
| caInfoList[].crlRefreshPeriod | Number | CRL更新周期(日) |
| caInfoList[].ocspActive | Boolean | OCSP有効化の有無 |
| caInfoList[].ocspRefreshPeriod | Number | OCSP更新周期(時間) |
| caInfoList[].ocspUrl | String | OCSPサーバーURL |
| totalCnt | Long | 全リポジトリ数 |
| totalPageNo | Long | 全ページ数 |
| currentPageNo | Long | 現在のページ番号(0から開始) |

### リポジトリ詳細照会

リポジトリ詳細情報を照会します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |

**必要権限**

- `VIEWER`以上

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| id | Long | リポジトリID |
| name | String | リポジトリ名 |
| description | String | リポジトリ説明 |
| toastProjectId | Long | NHN CloudプロジェクトID |
| templateCnt | Long | テンプレート数 |
| issuerCnt | Long | 発行者証明書数 |
| certificateCnt | Long | 全証明書数 |
| status | String | リポジトリ状態(`ACTIVE`, `INACTIVE`, `DELETED`) |
| crlActive | Boolean | CRL有効化の有無 |
| crlRefreshPeriod | Number | CRL更新周期(日) |
| ocspActive | Boolean | OCSP有効化の有無 |
| ocspRefreshPeriod | Number | OCSP更新周期(時間) |
| ocspUrl | String | OCSPサーバーURL |
| creationDatetime | LocalDateTime | 作成日時 |
| creationUser | String | 作成者 |
| lastChangeDatetime | LocalDateTime | 最終変更日時 |
| lastChangeUser | String | 最終変更者 |

### リポジトリ作成

リポジトリを作成します。

#### リクエスト

```
POST /v2.0/appkeys/{appkey}/ca-stores
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |

**Request Body**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| name | String | Y | リポジトリ名 | 最大64文字 |
| description | String | N | リポジトリ説明 | 最大256文字 |
| crlActive | Boolean | N | CRL有効化の有無 | デフォルト値: `false` |
| crlRefreshPeriod | Number | N | CRL更新周期(日) | 1 ～ 30<br>デフォルト値: `7` |
| ocspActive | Boolean | N | OCSP有効化の有無 | デフォルト値: `false` |
| ocspRefreshPeriod | Number | N | OCSP更新周期(時間) | 1 ～ 12<br>デフォルト値: `1` |

**必要権限**

- `ADMIN`

**リクエスト例**

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

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| id | Long | 作成されたリポジトリID |
| name | String | リポジトリ名 |
| toastProjectId | Long | NHN CloudプロジェクトID |
| status | String | リポジトリ状態 |
| creationDatetime | LocalDateTime | 作成日時 |
| creationUser | String | 作成者 |
| lastChangeDatetime | LocalDateTime | 最終変更日時 |
| lastChangeUser | String | 最終変更者 |

### リポジトリ修正

リポジトリを修正します。

#### リクエスト

```
PUT /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |

**Request Body**

リポジトリ作成と同じです。

**必要権限**

- `ADMIN`

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| id | Long | リポジトリID |
| name | String | リポジトリ名 |
| toastProjectId | Long | NHN CloudプロジェクトID |
| status | String | リポジトリ状態 |

### リポジトリ削除

リポジトリを削除します。

#### リクエスト

```
DELETE /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |

**必要権限**

- `ADMIN`

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| id | Long | 削除されたリポジトリID |
| name | String | リポジトリ名 |
| toastProjectId | Long | NHN CloudプロジェクトID |
| status | String | リポジトリ状態(`DELETED`) |

## 証明書(発行者)API

### 証明書一覧照会

リポジトリに含まれる証明書一覧を照会します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |

**Query Parameters**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| type | String | N | 証明書タイプフィルター | `all`, `issuer`, `leaf`<br>デフォルト値: `all` |
| page | Number | N | ページ番号 | 0から開始<br>デフォルト値: `0` |
| size | Number | N | ページサイズ | デフォルト値: `10` |
| search | String | N | 検索キーワード | Common Name基準 |

**必要権限**

- `VIEWER`以上

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| listCerts | Array | 証明書情報一覧 |
| listCerts[].id | Long | 証明書ID |
| listCerts[].name | String | 証明書名 |
| listCerts[].type | String | 証明書タイプ(`ROOT`, `INTERMEDIATE`, `LEAF`) |
| listCerts[].commonName | String | Common Name |
| listCerts[].serialNumber | String | シリアル番号(16進数、コロン区切り) |
| listCerts[].status | String | 証明書状態(`ACTIVE`, `INACTIVE`, `REVOKED`, `EXPIRED`, `DELETED`) |
| listCerts[].notBefore | LocalDateTime | 有効開始日 |
| listCerts[].notAfter | LocalDateTime | 有効期限 |
| listCerts[].isLeaf | Boolean | Leaf証明書かどうか |
| listCerts[].creationDatetime | LocalDateTime | 作成日時 |
| listCerts[].creationUser | String | 作成者 |
| totalCnt | Long | 全体数 |
| totalPageNo | Long | 全ページ数 |
| currentPageNo | Long | 現在のページ番号 |

### 証明書詳細照会

証明書詳細情報を照会します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |
| certId | Long | Y | 証明書ID |

**必要権限**

- `VIEWER`以上

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| certificateId | Long | 証明書ID |
| name | String | 証明書名 |
| description | String | 証明書説明 |
| type | String | 証明書タイプ(`ROOT`, `INTERMEDIATE`, `LEAF`) |
| status | String | 証明書状態 |
| commonName | String | Common Name |
| serialNumber | String | シリアル番号 |
| subjectInfo | Object | 主体情報 |
| notAfterDateTime | LocalDateTime | 有効期限 |
| notBeforeDateTime | LocalDateTime | 有効開始日 |
| keyAlgorithm | String | キーアルゴリズム(`RSA`, `EC`, `ED25519`) |
| keyLength | Number | キー長 |
| keyUsage | String[] | Key Usage一覧 |
| extendedKeyUsage | String[] | Extended Key Usage一覧 |
| certificatePem | String | 証明書PEM |
| chainCertificatePem | String | 証明書チェーンPEM |
| signatureAlgorithm | String | 署名アルゴリズム |
| isLeaf | Boolean | Leaf証明書かどうか |
| childCertificateList | Array | 下位証明書一覧 |
| crlUrl | String | CRLダウンロードURL |
| ocspUrl | String | OCSPサーバーURL |
| policies | String[] | Certificate Policies OID一覧 |

### 証明書(発行者)発行

ROOTまたはINTERMEDIATE証明書(発行者)を発行します。

#### リクエスト

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |

!!! danger "注意"
    ROOT、INTERMEDIATE証明書のみ発行可能です。LEAF証明書はテンプレートで発行する必要があります。

**Request Body**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| certificateType | String | Y | 証明書タイプ | `ROOT` または `INTERMEDIATE` |
| parentCertId | Number | 条件付き | 上位証明書ID | `certificateType`が`INTERMEDIATE`の場合は必須 |
| name | String | Y | 証明書名 | 最大64文字 |
| description | String | N | 証明書説明 | 最大256文字 |
| subjectInfo | Object | Y | 主体情報 | `commonName`必須 |
| ttlValue | Number | 条件付き | 有効期間(秒) | 0 ～ 315,360,000(最大10年)<br>specificDateと排他 |
| specificDate | String | 条件付き | 有効期限 | 1970-01-01T00:00:00 ～ 2999-12-31T23:59:59<br>形式: `2025-12-31T23:59:59`<br>ttlValueと排他 |
| backDateValidation | Number | N | バックデート有効性(秒) | 0 ～ 2,592,000(最大30日)<br>デフォルト値: `30` |
| maxDepth | Number | N | 下位CA作成可能深度 | -1 ～ 3<br>デフォルト値: `0` |
| keyInfo | Object | Y | キー情報 | 下記参照 |
| signatureAlgorithm | String | Y | 署名アルゴリズム | 下記参照<br>選択したKeyに合う署名アルゴリズムが必須(通常はSHA256形式を選択) |
| excludeCommonNameFromSans | Boolean | N | CNをSANから除外 | デフォルト値: `false` |
| sans | String[] | N | DNS SAN一覧 | |
| ipSans | String[] | N | IP SAN一覧 | |
| urlSans | String[] | N | URL SAN一覧 | |
| otherSans | OidInfo[] | N | その他SAN一覧 | 下記のOidInfoを参照 |

**KeyInfo**

| algorithm | keySize |
|-----------|---------|
| RSA | 2048, 3072, 4096 |
| ECまたはECDSA | 224、256、384、521 |
| ED25519 | 256(固定、値を入力しても無視される) |

**SignatureAlgorithm**

| アルゴリズム | 互換キー |
|----------|---------|
| SHA256_WITH_RSA | RSA |
| SHA384_WITH_RSA | RSA |
| SHA512_WITH_RSA | RSA |
| SHA256_WITH_ECDSA | ECまたはECDSA |
| SHA384_WITH_ECDSA | ECまたはECDSA |
| SHA512_WITH_ECDSA | ECまたはECDSA |
| ED25519 | ED25519 |

**SubjectInfo**

| 名前 | タイプ | 説明 | 制約条件 |
|------|------|------|-----------|
| commonName | String | Common Name(CN) | 最大64文字 |
| serialNumber | String | シリアル番号 | 最大64文字 |
| country | String | 国コード(C) | ISO 3166、2桁 |
| stateOrProvince | String | 州/都道府県(ST) | 最大128文字 |
| locality | String | 市区町村(L) | 最大128文字 |
| streetAddress | String | 住所(STREET) | 最大128文字 |
| postalCode | String | 郵便番号 | 最大40文字 |
| organization | String | 組織名(O) | 最大64文字 |
| organizationalUnit | String | 部署名(OU) | 最大64文字 |

**OidInfo(その他SAN)**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| oid | String | Y | OID(例: `1.2.840.113549.1.9.1`) |
| type | String | Y | `UTF8String`, `IA5String`, `PrintableString`, `BMPString`, `UniversalString` |
| value | String | Y | 値(最大255文字) |

**必要権限**

- `ADMIN`

**リクエスト例**

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

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| certificateId | Long | 発行された証明書ID |
| name | String | 証明書名 |
| type | String | 証明書タイプ(`ROOT`, `INTERMEDIATE`) |
| status | String | 証明書状態 |
| commonName | String | Common Name |
| serialNumber | String | シリアル番号 |
| notAfterDateTime | LocalDateTime | 有効期限 |
| notBeforeDateTime | LocalDateTime | 有効開始日 |
| keyAlgorithm | String | キーアルゴリズム |
| keyLength | Number | キー長 |
| certificatePem | String | 発行された証明書PEM |
| chainCertificatePem | String | 証明書チェーンPEM |
| signatureAlgorithm | String | 署名アルゴリズム |
| crlUrl | String | CRLダウンロードURL |
| ocspUrl | String | OCSPサーバーURL |

!!! note "参考"
    発行者証明書発行APIは、`privateKey`をレスポンスに含めません。発行者の秘密鍵はサーバーに安全に保存されます。

### 証明書失効

証明書を失効させます。

#### リクエスト

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/revoke
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |
| certId | Long | Y | 証明書ID |

**必要権限**

- `ADMIN`

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| result | Boolean | 失効処理の成否 |
| serialNumber | String | 失効された証明書のシリアル番号 |
| revocationDatetime | LocalDateTime | 失効日時 |

## テンプレートAPI

### テンプレート一覧照会

テンプレート一覧を照会します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |

**Query Parameters**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| page | Number | N | ページ番号 | 0から開始<br>デフォルト値: `0` |
| size | Number | N | ページサイズ | デフォルト値: `10` |
| search | String | N | 検索キーワード | テンプレート名 |

**必要権限**

- `VIEWER`以上

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| templates | Array | テンプレート情報一覧 |
| templates[].id | Long | テンプレートID |
| templates[].name | String | テンプレート名 |
| templates[].description | String | テンプレート説明 |
| totalCnt | Long | 全テンプレート数 |
| totalPageNo | Number | 全ページ数 |
| currentPageNo | Number | 現在のページ番号(0から開始) |

### テンプレート詳細照会

テンプレート詳細情報を照会します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |
| templateId | Long | Y | テンプレートID |

**必要権限**

- `VIEWER`以上

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| id | Long | テンプレートID |
| name | String | テンプレート名 |
| description | String | テンプレート説明 |
| subjectInfo | Object | 主体情報(テンプレート固定値) |
| keyAlgorithm | String | キーアルゴリズム |
| keyLength | Number | キー長 |
| keyInfo | Object | キー情報 |
| keyUsage | String[] | Key Usage一覧 |
| extendedKeyUsage | String[] | Extended Key Usage一覧 |
| extendedKeyUsageOids | String[] | Extended Key UsageカスタムOID一覧 |
| signatureAlgorithm | String | 署名アルゴリズム |
| signingCertificateId | Long | 署名に使用される発行者証明書ID |
| signingCertificateName | String | 署名に使用される発行者証明書名 |
| basicConstraintsValidForNonCa | Boolean | Non-CAに対するBasic Constraints検査の有無 |
| storeInServer | Boolean | サーバー保存の有無 |
| allowIpSans | Boolean | IP SAN許可の有無 |
| useCsrCommonName | Boolean | CSRのCN使用の有無 |
| useCsrSans | Boolean | CSRのSAN使用の有無 |
| useCsrOtherFields | Boolean | CSRのその他フィールド使用の有無 |
| maxTTL | String | 最大TTL(秒単位の文字列) |
| signatureBits | Number | 署名ビット数 |
| backDateValidation | Long | バックデート検証値(秒) |
| urlSansWhitelist | String[] | URL SANホワイトリスト |
| otherSansWhitelist | OidInfo[] | その他SANホワイトリスト |
| policies | String[] | ポリシーOID一覧 |
| otherFields | OidInfo[] | その他フィールド |
| creationDatetime | LocalDateTime | 作成日時 |
| creationUser | String | 作成者 |
| lastChangeDatetime | LocalDateTime | 最終変更日時 |
| lastChangeUser | String | 最終変更者 |

### テンプレート作成

テンプレートを作成します。

#### リクエスト

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |

**Request Body**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| name | String | Y | テンプレート名 | 最大64文字 |
| description | String | N | テンプレート説明 | 最大256文字 |
| parentCertId | Number | Y | 上位証明書ID | 発行者証明書のみ指定可能 |
| maxTTL | Number | 条件付き | 最大有効期間(秒) | 0 ～ 315,360,000(最大10年)<br>maxSpecificDateと排他 |
| maxSpecificDate | String | 条件付き | 最大有効期限制限 | 1970-01-01T00:00:00 ～ 2999-12-31T23:59:59<br>形式: `2025-12-31T23:59:59`<br>maxTTLと排他 |
| backDateValidation | Number | N | バックデート有効性(秒) | 0 ～ 2,592,000(最大30日)<br>デフォルト値: `30` |
| allowIpSans | Boolean | N | IP SAN許可の有無 | デフォルト値: `false` |
| urlSansWhitelist | String[] | N | URL SANホワイトリスト | |
| otherSansWhitelist | OidInfo[] | N | その他SANホワイトリスト | |
| storeInServer | Boolean | N | サーバーに証明書を保存するかどうか | デフォルト値: `true` |
| basicConstraintsValidForNonCa | Boolean | N | Non-CAに対するBasic Constraints検証 | デフォルト値: `false` |
| useCsrCommonName | Boolean | N | CSRのCN使用の有無 | デフォルト値: `false` |
| useCsrSans | Boolean | N | CSRのSAN使用の有無 | デフォルト値: `false` |
| keyInfo | Object | Y | キー情報 | 下記参照 |
| signatureBits | Number | N | 署名ビット数 | `256`、`384`、`512`<br>デフォルト値: `256`<br>ED25519の場合は無視される |
| keyUsage | String[] | N | キー使用用途 | 下記参照 |
| extendedKeyUsage | String[] | N | 拡張キー使用用途 | 下記参照 |
| extendedKeyUsageOids | String[] | N | 拡張キー使用カスタムOID | |
| policies | String[] | N | ポリシーOID一覧 | |
| subjectInfo | Object | N | 主体情報 | 下記参照 |
| useCsrOtherFields | Boolean | N | CSRのその他フィールド使用の有無 | デフォルト値: `false` |
| otherFields | OidInfo[] | N | その他フィールド | 下記参照 |

**KeyInfo**

| algorithm | keySize |
|-----------|---------|
| RSA | 2048, 3072, 4096 |
| ECまたはECDSA | 224、256、384、521 |
| ED25519 | 256(固定、値を入力しても無視される) |

**Key Usage値**

- `DIGITAL_SIGNATURE`
- `NON_REPUDIATION`
- `KEY_ENCIPHERMENT`
- `DATA_ENCIPHERMENT`
- `KEY_AGREEMENT`
- `KEY_CERT_SIGN`
- `CRL_SIGN`
- `ENCIPHER_ONLY`
- `DECIPHER_ONLY`

**Extended Key Usage値**

- `SERVER_AUTHENTICATION`
- `CLIENT_AUTHENTICATION`
- `CODE_SIGNING`
- `EMAIL_PROTECTION`
- `TIME_STAMPING`
- `OCSP_SIGNING`
- `ANY_EXTENDED_KEY_USAGE`

**SubjectInfo**

| 名前 | タイプ | 説明 | 制約条件 |
|------|------|------|-----------|
| serialNumber | String | シリアル番号 | 最大64文字 |
| country | String | 国コード(C) | ISO 3166、2桁 |
| stateOrProvince | String | 州/都道府県(ST) | 最大128文字 |
| locality | String | 市区町村(L) | 最大128文字 |
| streetAddress | String | 住所(STREET) | 最大128文字 |
| postalCode | String | 郵便番号 | 最大40文字 |
| organization | String | 組織名(O) | 最大64文字 |
| organizationalUnit | String | 部署名(OU) | 最大64文字 |

**OidInfo(その他フィールド)**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| oid | String | Y | OID(例: `1.2.840.113549.1.9.1`) |
| type | String | Y | `UTF8String`, `IA5String`, `PrintableString`, `BMPString`, `UniversalString` |
| value | String | Y | 値(最大255文字) |

**必要権限**

- `ADMIN`

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| templateId | Long | 作成されたテンプレートID |
| name | String | テンプレート名 |
| description | String | テンプレート説明 |
| signingCertificateId | Long | 署名に使用される発行者証明書ID |
| signingCertificateName | String | 署名に使用される発行者証明書名 |

### テンプレート修正

テンプレートを修正します。

#### リクエスト

```
PUT /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |
| templateId | Long | Y | テンプレートID |

**Request Body**

テンプレート作成と同じです。

**必要権限**

- `ADMIN`

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| body | Boolean | 修正の成否 |

### テンプレート削除

テンプレートを削除します。

#### リクエスト

```
DELETE /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |
| templateId | Long | Y | テンプレートID |

**必要権限**

- `ADMIN`

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| body | Boolean | 削除の成否 |

### テンプレートによる証明書発行

テンプレートを使用して証明書を発行します。

#### リクエスト

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/templates/{templateId}/certificates
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | リポジトリID |
| templateId | Long | Y | テンプレートID |

**Request Body**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| mode | String | Y | 証明書発行モード | `GENERATE`または`SIGN` |
| commonName | String | Y | Common Name | 最大64文字 |
| ttlValue | Number | 条件付き | 有効期間(秒) | 0 ～ 315,360,000(最大10年)<br>specificDateと排他 |
| specificDate | String | 条件付き | 特定の有効期限 | 1970-01-01T00:00:00 ～ 2999-12-31T23:59:59<br>形式: `2025-12-31T23:59:59`<br>ttlValueと排他 |
| csr | String | 条件付き | CSR | SIGNモード時は必須 |
| format | String | N | 証明書形式 | PEM, DER, PEM_BUNDLE<br>デフォルト値: `PEM`<br>無効な値を入力するとデフォルト値が適用される |
| privateKeyFormat | String | N | 秘密鍵形式 | PEM, DER, PKCS8<br>デフォルト値: `PEM`<br>無効な値を入力するとデフォルト値が適用される<br>GENERATEモード時のみ使用 |
| removeRootsFromChain | Boolean | N | チェーンからルートを削除 | SIGNモード時のみ使用 |
| excludeCommonNameFromSans | Boolean | N | CNをSANから除外 | |
| serialNumber | String | N | Serial Number | 最大64文字 |
| sans | String[] | N | DNS SAN一覧 | |
| ipSans | String[] | N | IP SAN一覧 | |
| urlSans | String[] | N | URL SAN一覧 | |
| otherSans | OidInfo[] | N | その他SAN一覧 | 下記参照 |

**OidInfo(その他SAN)**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| oid | String | Y | OID(例: `1.2.840.113549.1.9.1`) |
| type | String | Y | `UTF8String`, `IA5String`, `PrintableString`, `BMPString`, `UniversalString` |
| value | String | Y | 値(最大255文字) |

**必要権限**

- `ADMIN`

**リクエスト例(GENERATEモード)**

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

**リクエスト例(SIGNモード)**

```json
{
  "mode": "SIGN",
  "csr": "-----BEGIN CERTIFICATE REQUEST-----\nMIIC...\n-----END CERTIFICATE REQUEST-----",
  "ttlValue": 31536000,
  "format": "PEM",
  "removeRootsFromChain": false
}
```

#### レスポンス

**Response Body**(GENERATEモード)

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

**Response Body**(SIGNモード)

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

| フィールド | タイプ | 説明 |
|------|------|------|
| certificateId | Long | 発行された証明書ID |
| name | String | 証明書名 |
| type | String | 証明書タイプ(`LEAF`) |
| status | String | 証明書状態 |
| commonName | String | Common Name |
| serialNumber | String | シリアル番号 |
| notAfterDateTime | LocalDateTime | 有効期限 |
| notBeforeDateTime | LocalDateTime | 有効開始日 |
| keyAlgorithm | String | キーアルゴリズム |
| keyLength | Number | キー長 |
| certificatePem | String | 発行された証明書PEM |
| chainCertificatePem | String | 証明書チェーンPEM |
| signatureAlgorithm | String | 署名アルゴリズム |
| crlUrl | String | CRLダウンロードURL |
| ocspUrl | String | OCSPサーバーURL |
| privateKey | String | 秘密鍵(GENERATEモードの場合のみ返される) |

!!! danger "注意"
    GENERATEモードで発行する際、レスポンスに含まれる`privateKey`は**このレスポンスが唯一の返却時点**です。サーバーには保存されないため、ただちに安全な場所に保存してください。SIGNモードはクライアントが秘密鍵を保有するため、レスポンスに`privateKey`は含まれません。

## 証明書ダウンロードAPI

### 証明書のダウンロード

発行された証明書をPEM形式でダウンロードします。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{certId}/download
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caStoreId | Long | Y | リポジトリID |
| certId | Long | Y | 証明書ID |

**必要権限**

- `VIEWER`以上

#### レスポンス

**Response Headers**

- Content-Type: `application/x-pem-file`
- Content-Disposition: `attachment; filename={filename}.pem`

**Response Body**

証明書データ(PEM形式)

## CRL API

CRL(certificate revocation list)は、特定の発行者が発行した証明書のうち、失効した証明書のリストを提供するメカニズムです。クライアントはCRLをダウンロードして、証明書が失効しているか確認できます。

### CRL情報照会

特定発行者のCRL情報を照会します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caStoreId | Long | Y | リポジトリID |
| issuerCertId | Long | Y | 発行者証明書ID |

**必要権限**

- `VIEWER`以上

#### レスポンス

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

| フィールド | タイプ | 説明 |
|------|------|------|
| crlPem | String | CRL PEM形式データ |
| thisUpdate | LocalDateTime | CRL発行時間 |
| nextUpdate | LocalDateTime | 次回CRL予定時間 |

### CRLダウンロード(DER形式)

CRLをDER(バイナリ)形式でダウンロードします。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/der
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caStoreId | Long | Y | リポジトリID |
| issuerCertId | Long | Y | 発行者証明書ID |

**必要権限**

- 権限チェックなし(公開エンドポイント)

#### レスポンス

**Response Headers**

- Content-Type: `application/pkix-crl`
- Content-Disposition: `attachment; filename=crl_{caId}_{certId}.crl`

**Response Body**

CRLデータ(DER形式)

### CRLダウンロード(PEM形式)

CRLをPEM形式でダウンロードします。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl/pem
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caStoreId | Long | Y | リポジトリID |
| issuerCertId | Long | Y | 発行者証明書ID |

**必要権限**

- 権限チェックなし(公開エンドポイント)

#### レスポンス

**Response Headers**

- Content-Type: `application/pkix-crl`
- Content-Disposition: `attachment; filename=crl_{caId}_{certId}.pem`

**Response Body**

CRLデータ(PEM形式)

### CRL手動更新

CRLを手動で更新します。

#### リクエスト

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caStoreId | Long | Y | リポジトリID |
| issuerCertId | Long | Y | 発行者証明書ID |

**必要権限**

- `ADMIN`

#### レスポンス

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

OCSP(online certificate status protocol)は、個別証明書の失効状態を素早く確認できるプロトコルです。CRLとは異なり、全リストをダウンロードせず、特定証明書の状態のみリクエスト時点で照会できます。

### OCSP状態照会(GET)

Base64でエンコードされたOCSPリクエストを処理します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp/{ocspRequestBase64}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caStoreId | Long | Y | リポジトリID |
| ocspRequestBase64 | String | Y | Base64でエンコードされたOCSPリクエスト |

!!! danger "注意"
    OCSPリクエストをBase64でエンコードする際は、URL-safe形式に変換する必要があります。

    - 標準Base64エンコード後、次の文字を変換します：
        - `+` → `-`(plusをhyphenへ)
        - `/` → `_`(slashをunderscoreへ)
    - Padding文字(`=`)は削除します。
    - 例：
        - 変換前：`MEow/SDAwL+oGCC+sGAQUF/BzAh==`
        - 変換後：`MEow_SDAwL-oGCC-sGAQUF_BzAh`(`+` → `-`, `/` → `_`, `=` 削除)

**必要権限**

- 権限チェックなし(公開エンドポイント)

**リクエスト例**

```sh
# OCSPリクエスト生成及びBase64エンコード
OCSP_REQUEST=$(openssl ocsp -issuer ca.pem -cert cert.pem -reqout - | base64 -w 0)

# URLエンコードされたリクエスト送信
curl -X GET "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp/${OCSP_REQUEST}"
```

#### レスポンス

**Response Headers**

- Content-Type: `application/ocsp-response`

**Response Body**

OCSPレスポンス(DER形式)

### OCSP状態照会(POST)

DER形式OCSPリクエストを処理します。

#### リクエスト

```
POST /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/ocsp
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caStoreId | Long | Y | リポジトリID |

**必要権限**

- 権限チェックなし(公開エンドポイント)

**Request Headers**

- Content-Type: `application/ocsp-request`

**Request Body**

OCSPリクエスト(DER形式)

**リクエスト例**

```sh
# OCSPリクエスト生成
openssl ocsp -issuer ca.pem -cert cert.pem -reqout ocsp-request.der

# OCSPリクエスト送信
curl -X POST "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/ca-stores/1/ocsp" \
  -H "Content-Type: application/ocsp-request" \
  --data-binary @ocsp-request.der \
  -o ocsp-response.der

# OCSPレスポンス確認
openssl ocsp -respin ocsp-response.der -text
```

#### レスポンス

**Response Headers**

- Content-Type: `application/ocsp-response`

**Response Body**

OCSPレスポンス(DER形式)

## トラブルシューティング

### CRLが更新されない場合

1. コンソールのリポジトリ詳細情報でCRLが有効になっているか確認します。
2. 手動更新APIを呼び出して即時更新します。
3. CRL更新サイクル(`crlRefreshPeriod`)を確認し、調整します。

### OCSPレスポンスがない場合

1. コンソールのリポジトリ詳細情報でOCSPが有効になっているか確認します。
2. 正しいリポジトリIDを使用しているか確認します。
3. OCSPリクエストが正しい形式(DER)か確認します。

### OCSPレスポンス結果が実際の証明書状態と異なる場合

1. OCSPレスポンスは更新サイクルに従ってキャッシュされるため、最近証明書を失効させた場合、更新サイクルが過ぎるまで以前の状態が返されることがあります。
2. コンソールのリポジトリ詳細情報でOCSP更新サイクルを確認します。
3. 即時最新状態を確認する必要がある場合、更新サイクルが経過した後に再度照会します。

### 証明書にCRL/OCSP URLがない場合

- 証明書発行時に該当Extensionが含まれていない場合です。
- リポジトリ設定でCRL/OCSPを有効にした後、証明書を再発行する必要があります。

!!! danger "注意"
    - CRL/OCSP URLは証明書発行時点に含まれるため、設定変更後は証明書を再発行する必要があります。
    - すでに発行された証明書のCRL/OCSP URLは変更できません。
