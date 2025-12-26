# ACME証明書更新ガイド
**Management > Private CA > ACME証明書更新ガイド**

Private CAサービスはACME(automatic certificate management environment)プロトコルをサポートし、証明書の自動発行及び更新を可能にします。ACMEクライアント(例：Certbot)を使用すると、手動による介入なしに証明書を発行し、定期的に更新できます。

本ガイドでは、Private CA ACMEサーバーを利用してCertbotで証明書を発行する方法を案内します。

!!! tip 「ポイント」
    - **ACME(automatic certificate management environment)**：証明書の発行と更新を自動化する標準プロトコルです。
    - **Certbot**：Let's Encryptで開発されたACMEクライアントツールで、証明書を自動的に発行及び更新します。
    - **Base証明書**：ACME自動更新時に参照するテンプレートの役割を果たす証明書です。
    - **CSR(certificate signing request)**：証明書発行をリクエストするための署名リクエストファイルです。
    - **EAB(external account binding)**：ACMEサーバーに認証するためのアカウントバインディング情報です。

## 事前準備

ACMEによる証明書発行を開始する前に、次の事項を準備する必要があります。

### 1. Base証明書の発行

Base証明書は、ACMEサーバーが自動更新時に参照する「テンプレート」の役割を果たします。

- Base証明書に設定されたドメイン(CN、SAN)と同じドメインの証明書のみ、ACMEを通じて更新できます。
- Base証明書はコンソールで一般的な証明書発行手順で作成します。
- Base証明書を発行した後、該当証明書のIDをACME Directory URLに使用します。

### 2. Certbotのインストール

Certbotは最も広く使用されているACMEクライアントです。[Certbot公式ドキュメント](https://certbot.eff.org/)を参考に、OSに合わせてインストールします。

**インストール例(Ubuntu/Debian)：**

```bash
sudo apt update
sudo apt install certbot
```

**インストール例(CentOS/RHEL)：**

```bash
sudo yum install certbot
```

### 3. ACMEサーバー情報の確認

Private CAコンソールで次の情報を確認します。

- **ACME Directory URL**: `https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory`
- **ACMEトークンID**：コンソールで発行したACMEトークンID(**YOUR_ACME_TOKEN_ID**)
- **ACME HMACキー**：コンソールで発行したACMEトークンHMACキー(**YOUR_ACME_TOKEN_HMAC_KEY**)

## 証明書の更新

Certbotを使用して証明書を発行する手順は次のとおりです。

### コマンド構成

次は基本的な証明書発行コマンドの例です。

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

### 主なオプションの説明

| オプション | 説明 | 必須 | デフォルト値 |
|------|------|------|--------|
| `--manual` | Challenge検証を手動で実行します。ApacheやNginxの自動設定の代わりに手動モードを使用します。 | O | - |
| `--manual-auth-hook` | Challenge検証前に実行するスクリプトを指定します。空のスクリプトであっても更新時のエラー防止のために必要です。 | O | - |
| `--deploy-hook` | 証明書発行成功後に実行するスクリプトを指定します。証明書の移動、配布などの後処理に使用します。 | X | - |
| `--preferred-challenges` | Challenge方式を指定します(`http`、`dns`、`tls-alpn`)。一般的に`http`を使用します。 | O | - |
| `--server` | ACMEサーバーのDirectory URLを指定します。 | O | - |
| `-d` | 証明書に含めるドメインを指定します。Base証明書のCNとSANをコンソールで確認し、正確に入力する必要があります。 | O | - |
| `--eab-kid` | External Account BindingのキーIDです。Private CAで発行したACMEトークンIDを入力します。 | O | - |
| `--eab-hmac-key` | EAB HMACキー(Base64エンコード)です。Private CAで発行したACMEトークンHMACキーを入力します。 | O | - |
| `--key-type` | 秘密鍵タイプ(`rsa`、`ecdsa`)を指定します。 | X | rsa |
| `--rsa-key-size` | RSA鍵長を指定します。`--key-type rsa`の場合にのみ使用し、`2048`、`3072`、`4096`の値を使用できます。 | X | 2048 |
| `--elliptic-curve` | ECDSA鍵曲線を指定します。`--key-type ecdsa`の場合にのみ使用し、`secp256r1`、`secp384r1`、`secp521r1`の値を使用できます。 | X | secp256r1 |
| `--agree-tos` | サービス利用規約に自動的に同意します。インタラクティブな入力を回避します。 | X | - |
| `--register-unsafely-without-email` | メールアドレスなしでアカウントを登録します。インタラクティブな入力を回避します。 | X | - |

!!! tip 「ポイント」
    - `--manual`モードは手動Challenge検証に使用されます。
    - `manual-auth-hook`は証明書更新時の安定性確保のために必ず必要です。
    - `deploy-hook`を活用すると、証明書発行後に自動的に配布及び後処理作業を実行できます。
    - `--server`、`--eab-kid`、`--eab-hmac-key`はPrivate CA ACMEサーバー連携に必須です。

!!! danger "注意"
    ドメイン指定時、Base証明書に設定されたCN(common name)とドメインSAN(subject alternative name)を正確に入力する必要があります。証明書発行前にコンソールでBase証明書のCNとSAN情報を確認し、`-d`オプションに正しいドメインを指定したか必ず検証してください。

### Hookスクリプト例

#### pre.sh(認証前実行スクリプト)

`manual-auth-hook`は内容が空であってもファイルとして存在する必要があります。これは証明書更新(renew)時のエラーを防止するためです。

```bash
#!/bin/bash
# 必要な認証前処理作業をここに追加できます。
```

#### post.sh(証明書発行後実行スクリプト)

`deploy-hook`は証明書が正常に発行された場合にのみ実行されます。

```bash
#!/bin/bash
# 証明書発行完了後の処理作業
echo "[post.sh] 証明書発行完了！"

# 証明書を任意の場所にコピー
cp /etc/letsencrypt/live/example.com/fullchain.pem ~/Downloads/
cp /etc/letsencrypt/live/example.com/privkey.pem ~/Downloads/

# Webサーバー再起動などの追加作業
# systemctl reload nginx
```

## 発行された証明書の確認

証明書は基本的に次のパスに保存されます。

```
/etc/letsencrypt/live/<ドメイン名(CN)>/
├── cert.pem          # サーバー証明書
├── chain.pem         # 中間証明書チェーン
├── fullchain.pem     # cert.pem + chain.pem
└── privkey.pem       # 秘密鍵
```

### 証明書内容の確認

```bash
# 証明書情報の確認
openssl x509 -in /etc/letsencrypt/live/<ドメイン名(CN)>/cert.pem -text -noout

# 証明書有効期間の確認
openssl x509 -in /etc/letsencrypt/live/<ドメイン名(CN)>/cert.pem -noout -dates
```

## 証明書自動更新の設定

Certbotは期限切れが近い証明書を自動的に更新できます。

### 更新前提条件

- 初回発行履歴が必要です。
- `/etc/letsencrypt/renewal/<ドメイン>.conf`ファイルが存在する必要があります。
- `/etc/letsencrypt/live/<ドメイン>/`ディレクトリに証明書ファイルが存在する必要があります。

### 自動更新設定

Certbotインストール時、自動的にcronまたはsystemd timerが登録され、定期的に証明書の期限切れ可否を確認します。

**基本更新サイクル**：期限切れ30日前から自動更新試行

### 手動更新

必要に応じて次のコマンドで手動更新を実行できます。

```bash
# 全ての証明書更新試行
sudo certbot renew

# 強制更新
sudo certbot renew --force-renewal
```

### Cronジョブ登録

自動更新が登録されていない場合、手動でcronジョブを追加できます。

```bash
# crontab編集
sudo crontab -e

# 毎日午前2時に更新確認
0 2 * * * certbot renew --no-random-sleep-on-renew
```

### 更新サイクル変更

`/etc/letsencrypt/renewal/<ドメイン>.conf`ファイルで更新サイクルを調整できます。

```ini
[renewalparams]
renew_before_expiry = 30 days
```

`renew_before_expiry`値を変更して、証明書期限切れの何日前から更新を試行するか設定できます。

!!! danger "注意"
    - ED25519鍵タイプは現在Certbotでサポートしていません。証明書チェーンにED25519アルゴリズムを使用する証明書が含まれている場合、発行及び更新はできません。
    - 証明書チェーンは最低3段以上(Root → Intermediate → Leaf)で構成する必要があります。
    - `renew_before_expiry`値はACMEサーバーのRate Limitポリシーを考慮して慎重に調整する必要があります。
    - Certbotインストール時に自動登録されたcronジョブにはデフォルトオプションが含まれている場合があるため、必要に応じて`/etc/cron.d/certbot`ファイルを修正することを推奨します。
    - デフォルトcronジョブにはランダム遅延(`perl -e 'sleep int(rand(43200))'`)が含まれています。これはACMEサーバーの過負荷防止のためであり、即時実行が必要な場合は該当構文を削除するか`--no-random-sleep-on-renew`オプションを使用する必要があります。

## トラブルシューティング

### 証明書発行失敗時

1. **ACME Directory URL確認**：`--server`オプションのURLが正しいか確認します。
2. **EAB認証情報確認**：`--eab-kid`と`--eab-hmac-key`の値が正確か確認します。
3. **ドメイン検証失敗**：Challenge方式が環境に合っているか確認します。HTTP Challengeの場合、80番ポートが開いている必要があります。
4. **Hookスクリプト権限**：`pre.sh`と`post.sh`ファイルに実行権限があるか確認します。

### 証明書更新失敗時

1. **Renewal設定確認**：`/etc/letsencrypt/renewal/<ドメイン>.conf`ファイルが存在し、正しいか確認します。
2. **Hookスクリプト存在確認**：`manual-auth-hook`で指定したスクリプトがまだ存在するか確認します。

## ACMEプロトコル情報

Private CAで提供するACME Directory URL(`/directory`)を通じて、ACMEクライアントは必要な全てのエンドポイント情報を自動的に取得します。

ACMEプロトコルの全体フローはクライアントによって自動的に処理されるため、ユーザーはDirectory URLのみ提供すればよいです。

ACMEプロトコルの詳細については、[RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555)を参照してください。

## 参考資料

- [Certbot公式ドキュメント](https://certbot.eff.org/)
- [ACMEプロトコル仕様(RFC 8555)](https://datatracker.ietf.org/doc/html/rfc8555)
- [Let's Encrypt - Challenge Types](https://letsencrypt.org/docs/challenge-types/)
