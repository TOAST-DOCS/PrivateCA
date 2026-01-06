# API v2.0ガイド
**Management > Private CA > API v2.0ガイド**

NHN Cloud Private CA APIを使用して証明書をプログラムで管理できます。

## 基本情報

### API Endpoint

| リージョン | エンドポイント |
| --- | --- |
| KR1 | https://kr1-pca.api.nhncloudservice.com |

### API一覧

| Method | URI | 説明 |
|--------|-----|------|
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{certId}/download | PEM形式の証明書をダウンロードします。 |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl | CRL情報(PEM、発行/更新時間)を照会します。 |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/der | DER形式のCRLをダウンロードします。 |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/pem | PEM形式のCRLをダウンロードします。 |
| POST | /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl | CRLを手動で更新します。 |
| GET | /v2.0/appkeys/{appkey}/cas/{caId}/ocsp/{ocspRequestBase64} | Base64エンコードされたOCSPリクエストで証明書の状態を照会します。 |
| POST | /v2.0/appkeys/{appkey}/cas/{caId}/ocsp | DER形式のOCSPリクエストで証明書の状態を照会します。 |

## 事前準備

### 認証

全てのAPIリクエストには、次のような認証ヘッダが必要です。

```
X-NHN-Authorization: Bearer {access_token}
```

!!! tip 「ポイント」
    - 認証ヘッダに必要な認証トークンの詳細については、[こちら](https://docs.nhncloud.com/ja/nhncloud/ja/public-api/api-authentication/)で詳しく確認できます。
    - アプリキー(appkey)はコンソールで確認でき、全てのAPIパスに含まれている必要があります。

### 権限管理

Private CA APIはロールベースアクセス制御(RBAC)を使用し、次のように区分されます。

- **VIEWER**：証明書のダウンロード、CRL照会などの読み取り作業のみ実行できます。
- **ADMIN**：CRL手動更新など、全ての管理作業を実行できます。
- **公開エンドポイント**：CRLダウンロード(DER/PEM)とOCSP APIは、証明書検証のために認証なしでアクセス可能です。

### 証明書形式

Private CA APIで使用する主な証明書形式は次のとおりです。

- **PEM(privacy enhanced mail)**：テキストベースの証明書形式で、Base64でエンコードされており、`-----BEGIN CERTIFICATE-----`で始まります。人間が読むことができ、編集が容易です。
- **DER(distinguished encoding rules)**：バイナリ形式の証明書で、PEMよりファイルサイズが小さく効率的です。主にJavaアプリケーションで使用されます。

## 証明書ダウンロードAPI

### 証明書のダウンロード

発行された証明書をPEM形式でダウンロードします。

#### リクエスト

```
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{certId}/download
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caId | Long | Y | リポジトリID |
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
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caId | Long | Y | リポジトリID |
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
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/der
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caId | Long | Y | リポジトリID |
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
GET /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl/pem
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caId | Long | Y | リポジトリID |
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
POST /v2.0/appkeys/{appkey}/cas/{caId}/certs/{issuerCertId}/crl
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caId | Long | Y | リポジトリID |
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
GET /v2.0/appkeys/{appkey}/cas/{caId}/ocsp/{ocspRequestBase64}
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caId | Long | Y | リポジトリID |
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
curl -X GET "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/cas/1/ocsp/${OCSP_REQUEST}"
```

#### レスポンス

**Response Headers**

- Content-Type: `application/ocsp-response`

**Response Body**

OCSPレスポンス(DER形式)

### OCSP状態照会(POST)

DER形式のOCSPリクエストを処理します。

#### リクエスト

```
POST /v2.0/appkeys/{appkey}/cas/{caId}/ocsp
```

**Path Parameters**

| 名前 | タイプ | 必須 | 説明 |
|------|------|------|------|
| appkey | String | Y | アプリキー |
| caId | Long | Y | リポジトリID |

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
curl -X POST "https://kr1-pca.api.nhncloudservice.com/v2.0/appkeys/my-appkey/cas/1/ocsp" \
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
