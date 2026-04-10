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

#### ストア

| Method | URI | 説明 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/ca-stores | ストア一覧を照会 |
| POST | /v2.0/appkeys/{appkey}/ca-stores | ストアを作成 |
| GET | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | ストア詳細情報を照会 |
| PUT | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | ストアを修正 |
| DELETE | /v2.0/appkeys/{appkey}/ca-stores/{caStoreId} | ストアを削除 |

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

- **VIEWER**: ストア/証明書/テンプレートの照会、証明書ダウンロード、CRL照会などの読み取り操作のみ実行できます。
- **ADMIN**: ストア/証明書/テンプレートの作成・修正・削除、CRL手動更新など、すべての管理操作を実行できます。
- **パブリックエンドポイント**: CRLダウンロード(DER/PEM)とOCSP APIは、証明書検証用として認証なしでアクセス可能です。

### 証明書形式

Private CA APIで使用する主な証明書形式は次のとおりです。

- **PEM(privacy enhanced mail)**：テキストベースの証明書形式で、Base64でエンコードされており、`-----BEGIN CERTIFICATE-----`で始まります。人間が読むことができ、編集が容易です。
- **DER(distinguished encoding rules)**：バイナリ形式の証明書で、PEMよりファイルサイズが小さく効率的です。主にJavaアプリケーションで使用されます。

## ストアAPI

### ストア一覧照会

ストア一覧を照会します。

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
| search | String | N | 検索キーワード | ストア名または説明 |

**必要権限**

- `VIEWER`以上

### ストア詳細照会

ストア詳細情報を照会します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | ストアID |

**必要権限**

- `VIEWER`以上

### ストア作成

ストアを作成します。

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
| name | String | Y | ストア名 | 最大64文字 |
| description | String | N | ストア説明 | 最大256文字 |
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

### ストア修正

ストアを修正します。

#### リクエスト

```
PUT /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | ストアID |

**Request Body**

ストア作成と同じです。

**必要権限**

- `ADMIN`

### ストア削除

ストアを削除します。

#### リクエスト

```
DELETE /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | ストアID |

**必要権限**

- `ADMIN`

## 証明書(発行者)API

### 証明書一覧照会

ストアに含まれる証明書一覧を照会します。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/ca-stores/{caStoreId}/certs
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | Appkey |
| caStoreId | Long | Y | ストアID |

**Query Parameters**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| type | String | N | 証明書タイプフィルター | `all`, `issuer`, `leaf`<br>デフォルト値: `all` |
| page | Number | N | ページ番号 | 0から開始<br>デフォルト値: `0` |
| size | Number | N | ページサイズ | デフォルト値: `10` |
| search | String | N | 検索キーワード | Common Name基準 |

**必要権限**

- `VIEWER`以上

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
| caStoreId | Long | Y | ストアID |
| certId | Long | Y | 証明書ID |

**必要権限**

- `VIEWER`以上

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
| caStoreId | Long | Y | ストアID |

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
| caStoreId | Long | Y | ストアID |
| certId | Long | Y | 証明書ID |

**必要権限**

- `ADMIN`

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
| caStoreId | Long | Y | ストアID |

**Query Parameters**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| page | Number | N | ページ番号 | 0から開始<br>デフォルト値: `0` |
| size | Number | N | ページサイズ | デフォルト値: `10` |
| search | String | N | 検索キーワード | テンプレート名 |

**必要権限**

- `VIEWER`以上

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
| caStoreId | Long | Y | ストアID |
| templateId | Long | Y | テンプレートID |

**必要権限**

- `VIEWER`以上

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
| caStoreId | Long | Y | ストアID |

**Request Body**

| 名前 | タイプ | 必須 | 説明 | 制約条件 |
|------|------|------|------|-----------|
| name | String | Y | テンプレート名 | 最大64文字 |
| description | String | N | テンプレート説明 | 最大256文字 |
| parentCertId | Number | Y | 上位証明書ID | |
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
| caStoreId | Long | Y | ストアID |
| templateId | Long | Y | テンプレートID |

**Request Body**

テンプレート作成と同じです。

**必要権限**

- `ADMIN`

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
| caStoreId | Long | Y | ストアID |
| templateId | Long | Y | テンプレートID |

**必要権限**

- `ADMIN`

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
| caStoreId | Long | Y | ストアID |
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
