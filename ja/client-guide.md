# ACME証明書更新ガイド(Certbot, acme.sh)
**Management > Private CA > ACME証明書更新ガイド(Certbot, acme.sh)**

Private CAサービスは、ACME(automatic certificate management environment)プロトコルに対応し、証明書の自動発行及び更新を可能にします。ACMEクライアント(例: Certbot, acme.sh)を使用すると、手動での操作なしに証明書を発行し、定期的に更新できます。

本ガイドでは、Private CA ACMEサーバーを利用してCertbotまたはacme.shで証明書を発行し、自動的に更新する方法を案内します。

!!! tip 「ポイント」
    - **ACME(automatic certificate management environment)**: 証明書の発行と更新を自動化する標準プロトコルです。
    - **Certbot**: Let's Encryptで開発されたACMEクライアントツールで、証明書を自動的に発行及び更新します。
    - **acme.sh**: 純粋なUnix Shellで作成された軽量ACMEクライアントで、証明書を自動的に発行及び更新します。
    - **Base証明書**: ACME自動更新時に参照するテンプレート役割の証明書です。
    - **CSR(certificate signing request)**: 証明書の発行をリクエストするための署名リクエストファイルです。
    - **EAB(external account binding)**: ACMEサーバーに認証するためのアカウントバインディング情報です。

## 事前準備

ACMEを利用した証明書発行を開始する前に、以下の事項を準備する必要があります。

### 1. Base証明書の発行

Base証明書は、ACMEサーバーが自動更新時に参照する「テンプレート」の役割を果たします。

- Base証明書に設定されたドメイン(CN, SAN)と同一ドメインの証明書のみ、ACMEを利用して更新できます。
- Base証明書は、コンソールで一般的な証明書発行手順により作成します。
- Base証明書を発行した後、該当証明書のIDをACME Directory URLに使用します。

### 2. ACMEサーバー情報の確認

Private CAコンソールで以下の情報を確認します。

- **ACME Directory URL**: `https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory`
- **ACMEトークンID**: コンソールで発行したACMEトークンID(**YOUR_ACME_TOKEN_ID**)
- **ACME HMACキー**: コンソールで発行したACMEトークンHMACキー(**YOUR_ACME_TOKEN_HMAC_KEY**)

## 証明書の更新

ACMEクライアントとしてCertbotまたはacme.shを使用できます。使用環境に合わせてツールを選択して進めてください。

- **Certbot**: Let's Encryptで開発された広く使用されているACMEクライアントで、Pythonベースです。
- **acme.sh**: 純粋なShellスクリプトで作成された軽量ACMEクライアントで、依存関係が少なくインストールが簡単です。

!!! tip 「ポイント」
    Kubernetes環境で証明書を自動的に管理するには、[ACME証明書更新ガイド(cert-manager)](cert-manager-guide.md)を参照してください。

### Certbotを利用した証明書更新

#### Certbotのインストール

Certbotは最も広く使用されているACMEクライアントです。[Certbot公式ドキュメント](https://certbot.eff.org/)を参照して、OSに合わせてインストールします。

**インストール例(Ubuntu/Debian)**

```bash
sudo apt update
sudo apt install certbot
```

**インストール例(CentOS/RHEL)**

```bash
sudo yum install certbot
```

#### コマンド構成

以下は基本的な証明書発行コマンドの例です。

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

#### 主なオプションの説明

| オプション | 説明 | 必須 | デフォルト値 |
|------|------|------|--------|
| `--manual` | Challenge検証を手動で実行します。ApacheやNginxの自動設定の代わりに手動モードを使用します。 | O | - |
| `--manual-auth-hook` | Challenge検証前に実行するスクリプトを指定します。空のスクリプトでも更新時のエラー防止のために必要です。 | O | - |
| `--deploy-hook` | 証明書発行成功後に実行するスクリプトを指定します。証明書の移動、配布などの後処理に使用します。 | X | - |
| `--preferred-challenges` | Challenge方式を指定します(`http`、`dns`、`tls-alpn`)。一般的に`http`を使用します。 | O | - |
| `--server` | ACMEサーバーのDirectory URLを指定します。 | O | - |
| `-d` | 証明書に含めるドメインを指定します。Base証明書のCNとSANをコンソールで確認し、正確に入力する必要があります。 | O | - |
| `--eab-kid` | External Account BindingのキーIDです。Private CAで発行したACMEトークンIDを入力します。 | O | - |
| `--eab-hmac-key` | EAB HMACキー(Base64エンコーディング)です。Private CAで発行したACMEトークンHMACキーを入力します。 | O | - |
| `--key-type` | 秘密鍵のタイプ(`rsa`、`ecdsa`)を指定します。 | X | rsa |
| `--rsa-key-size` | RSAキーの長さを指定します。`--key-type rsa`の場合にのみ使用し、`2048`、`3072`、`4096`の値を使用できます。 | X | 2048 |
| `--elliptic-curve` | ECDSAキー曲線を指定します。`--key-type ecdsa`の場合にのみ使用し、`secp256r1`、`secp384r1`、`secp521r1`の値を使用できます。 | X | secp256r1 |
| `--agree-tos` | サービス利用規約に自動的に同意します。インタラクティブな入力を防止します。 | X | - |
| `--register-unsafely-without-email` | メールアドレスなしでアカウントを登録します。インタラクティブな入力を防止します。 | X | - |

!!! tip 「ポイント」
    - `--manual`モードは手動Challenge検証に使用されます。
    - `manual-auth-hook`は、証明書更新時の安定性を確保するために必ず必要です。
    - `deploy-hook`を活用すると、証明書発行後に自動的に配布及び後処理作業を実行できます。
    - `--server`、`--eab-kid`、`--eab-hmac-key`は、Private CA ACMEサーバーとの連携に必須です。

!!! danger "注意"
    ドメイン指定時、Base証明書に設定されたCN(common name)とドメインSAN(subject alternative name)を正確に入力する必要があります。証明書発行前にコンソールでBase証明書のCNとSAN情報を確認し、`-d`オプションに正しいドメインを指定しているか必ず検証してください。

#### Hookスクリプトの例

##### pre.sh(認証前実行スクリプト)

`manual-auth-hook`は内容が空であってもファイルとして存在する必要があります。これは証明書更新(renew)時のエラーを防ぐためです。

```bash
#!/bin/bash
# 必要な認証前処理作業をここに追加できます。
```

##### post.sh(証明書発行後実行スクリプト)

`deploy-hook`は証明書が正常に発行された場合にのみ実行されます。

```bash
#!/bin/bash
# 証明書発行完了後の処理作業
echo "[post.sh] 証明書発行完了！"

# 証明書を任意の場所にコピー
cp /etc/letsencrypt/live/example.com/fullchain.pem ～/Downloads/
cp /etc/letsencrypt/live/example.com/privkey.pem ～/Downloads/

# Webサーバーの再起動など追加作業
# systemctl reload nginx
```

#### 発行された証明書の確認

証明書は基本的に以下のパスに保存されます。

```
/etc/letsencrypt/live/<ドメイン名(CN)>/
├── cert.pem          # サーバー証明書
├── chain.pem         # 中間証明書チェーン
├── fullchain.pem     # cert.pem + chain.pem
└── privkey.pem       # 秘密鍵
```

##### 証明書内容の確認

```bash
# 証明書情報の確認
openssl x509 -in /etc/letsencrypt/live/<ドメイン名(CN)>/cert.pem -text -noout

# 証明書有効期限の確認
openssl x509 -in /etc/letsencrypt/live/<ドメイン名(CN)>/cert.pem -noout -dates
```

#### 証明書自動更新の設定

Certbotは満了が近づいた証明書を自動的に更新できます。

##### 更新の前提条件

- 初回発行履歴が必要です。
- `/etc/letsencrypt/renewal/<ドメイン>.conf`ファイルが存在する必要があります。
- `/etc/letsencrypt/live/<ドメイン>/`ディレクトリに証明書ファイルが存在する必要があります。

##### 自動更新設定

Certbotインストール時に自動的にcronまたはsystemd timerが登録され、定期的に証明書の有効期限を確認します。

**基本更新周期**: 満了30日前から自動更新を試行

##### 手動更新

必要に応じて次のコマンドで手動更新を実行できます。

```bash
# 全ての証明書更新試行
sudo certbot renew

# 強制更新
sudo certbot renew --force-renewal
```

##### Cronジョブの登録

自動更新が登録されていない場合、手動でcronジョブを追加できます。

```bash
# crontab編集
sudo crontab -e

# 毎日午前2時に更新確認
0 2 * * * certbot renew --no-random-sleep-on-renew
```

##### 更新周期の変更

`/etc/letsencrypt/renewal/<ドメイン>.conf`ファイルで更新周期を調整できます。

```ini
[renewalparams]
renew_before_expiry = 30 days
```

`renew_before_expiry`値を変更して、証明書満了の何日前から更新を試みるか設定できます。

!!! danger "注意"
    - ED25519キータイプは現在Certbotでサポートしていません。証明書チェーンにED25519アルゴリズムを使用する証明書が含まれている場合、発行及び更新はできません。
    - 証明書チェーンは、最低3段階以上(Root → Intermediate → Leaf)で構成される必要があります。
    - `renew_before_expiry`値は、ACMEサーバーのRate Limitポリシーを考慮して慎重に調整する必要があります。
    - Certbotインストール時に自動登録されたcronジョブには基本オプションが含まれている場合があるため、必要に応じて`/etc/cron.d/certbot`ファイルを修正することが推奨されます。
    - 基本cronジョブにはランダム遅延(`perl -e 'sleep int(rand(43200))'`)が含まれています。これはACMEサーバーの過負荷防止のためであり、即時実行が必要な場合は該当構文を削除するか`--no-random-sleep-on-renew`オプションを使用する必要があります。

#### トラブルシューティング

##### 証明書発行失敗時

1. **ACME Directory URL確認**: `--server`オプションのURLが正しいか確認します。
2. **EAB認証情報確認**: `--eab-kid`と`--eab-hmac-key`の値が正確か確認します。
3. **ドメイン検証失敗**: Challenge方式が環境に合っているか確認します。HTTP Challengeの場合、80番ポートが開いている必要があります。
4. **Hookスクリプト権限**: `pre.sh`と`post.sh`ファイルに実行権限があるか確認します。

##### 証明書更新失敗時

1. **Renewal設定確認**: `/etc/letsencrypt/renewal/<ドメイン>.conf`ファイルが存在し、正しいか確認します。
2. **Hookスクリプト存在確認**: `manual-auth-hook`で指定したスクリプトが依然として存在するか確認します。

### acme.shを利用した証明書更新

acme.shは、純粋なUnix Shellで作成された軽量ACMEクライアントです。依存関係が少なくインストールが簡単で、様々な環境で使用できます。

#### acme.shのインストール

acme.shはインストールスクリプトを通じて簡単にインストールできます。

**インストール**

```bash
curl https://get.acme.sh | sh
```

インストールが完了したら環境変数をロードします。

```bash
. ～/.acme.sh/acme.sh.env
acme.sh --version
```

!!! tip 「ポイント」
    - コンテナや新しいシェルセッションに再接続する際、`. ~/.acme.sh/acme.sh.env`コマンドで環境変数をリロードする必要があります。
    - インストール時、メールアドレスは任意です。

**インストール例(手動ダウンロード)**

```bash
git clone https://github.com/acmesh-official/acme.sh.git
cd acme.sh
./acme.sh --install
```

#### ACMEアカウント登録

証明書を発行する前に、まずACMEサーバーにアカウントを登録する必要があります。

```bash
acme.sh --register-account \
  --server https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory \
  --eab-kid "YOUR_ACME_TOKEN_ID" \
  --eab-hmac-key "YOUR_ACME_TOKEN_HMAC_KEY"
```

!!! danger "注意"
    - アカウント登録は**初回1回のみ**実行します。
    - 登録後は、証明書発行時に`--eab-kid`と`--eab-hmac-key`オプションを省略できます。

#### 証明書の発行

アカウント登録後、証明書を発行できます。

**基本発行コマンドの例**

```bash
acme.sh --issue \
  -d example.com -d www.example.com \
  --server https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory \
  --keylength 2048 \
  --standalone
```

#### 主なオプションの説明

| オプション | 説明 | 必須 | デフォルト値 |
|------|------|------|--------|
| `--register-account` | ACMEサーバーにアカウントを登録します。初回1回のみ実行します。 | O(初回) | - |
| `--issue` | 証明書を発行します。 | O | - |
| `--server` | ACMEサーバーのDirectory URLを指定します。 | O | - |
| `--eab-kid` | External Account BindingのキーIDです。Private CAで発行したACMEトークンIDを入力します。アカウント登録時にのみ必要です。 | O(登録時) | - |
| `--eab-hmac-key` | EAB HMACキー(Base64エンコーディング)です。Private CAで発行したACMEトークンHMACキーを入力します。アカウント登録時にのみ必要です。 | O(登録時) | - |
| `-d` | 証明書に含めるドメインを指定します。Base証明書のCNとSANをコンソールで確認し、正確に入力する必要があります。複数指定可能です。 | O | - |
| `--standalone` | StandaloneモードでHTTP-01 Challengeを処理します。acme.shが一時的なWebサーバーを実行します。 | O | - |
| `--httpport` | HTTP-01 Challengeに使用するポート番号を指定します。 | X | 80 |
| `--keylength` | 秘密鍵の長さを指定します。RSAは`2048`、`3072`、`4096`、ECDSAは`ec-256`、`ec-384`、`ec-521`の値を使用できます。 | X | 2048 |
| `--debug` | デバッグモードで実行して詳細ログを出力します。 | X | - |

!!! tip 「ポイント」
    - `--standalone`モードはacme.shが一時的なWebサーバーを実行してChallengeを処理します。80番ポートを使用できる必要があります。
    - アカウント登録後は、`--eab-kid`と`--eab-hmac-key`オプションなしで証明書を発行できます。

!!! danger "注意"
    ドメイン指定時、Base証明書に設定されたCN(common name)とドメインSAN(subject alternative name)を正確に入力する必要があります。証明書発行前にコンソールでBase証明書のCNとSAN情報を確認し、`-d`オプションに正しいドメインを指定しているか必ず検証してください。

#### Standaloneモードでの証明書発行

acme.shが一時的なWebサーバーを実行してHTTP-01 Challengeを処理します。

```bash
acme.sh --issue \
  -d example.com \
  --server https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory \
  --keylength 2048 \
  --standalone
```

!!! danger "注意"
    - 80番ポートが開いており、使用可能である必要があります。
    - 既存のWebサーバーが80番ポートを使用中の場合、一時的に中断する必要があります。
    - 他のポートを使用するには`--httpport`オプションを追加します。

#### 発行された証明書の確認

証明書は基本的に以下のパスに保存されます。

```
～/.acme.sh/<ドメイン名(CN)>/
├── <ドメイン名>.cer        # サーバー証明書(リーフ証明書)
├── <ドメイン名>.key        # 秘密鍵
├── ca.cer                # CAチェーン(Intermediate + Root)
└── fullchain.cer         # 全体チェーン(リーフ証明書 + CAチェーン)
```

!!! tip 「ポイント」
    - `<ドメイン名>.cer`: リーフ証明書のみ含む
    - `fullchain.cer`: リーフ証明書 + 中間証明書 + ルート証明書の全体チェーン
    - `ca.cer`: CAチェーン(中間証明書 + ルート証明書)

##### 証明書内容の確認

```bash
# 証明書情報の確認
openssl x509 -in ～/.acme.sh/example.com/example.com.cer -text -noout

# 証明書有効期限の確認
openssl x509 -in ～/.acme.sh/example.com/example.com.cer -noout -dates
```

#### 証明書のインストール(配布)

acme.shは`--install-cert`コマンドで証明書を任意の場所にコピーし、Webサーバーを再起動できます。

**Nginxの例**

```bash
acme.sh --install-cert -d example.com \
  --key-file       /etc/nginx/ssl/privkey.pem \
  --cert-file      /etc/nginx/ssl/cert.pem \
  --fullchain-file /etc/nginx/ssl/fullchain.pem \
  --ca-file        /etc/nginx/ssl/ca.pem \
  --reloadcmd      "nginx -s reload"
```

**Apacheの例**

```bash
acme.sh --install-cert -d example.com \
  --key-file       /etc/apache2/ssl/privkey.pem \
  --cert-file      /etc/apache2/ssl/cert.pem \
  --fullchain-file /etc/apache2/ssl/fullchain.pem \
  --ca-file        /etc/apache2/ssl/ca.pem \
  --reloadcmd      "systemctl reload apache2"
```

!!! tip 「ポイント」
    - `--key-file`: 秘密鍵ファイルの保存パス
    - `--cert-file`: リーフ証明書の保存パス
    - `--fullchain-file`: 全体チェーン(リーフ + CAチェーン)の保存パス
    - `--ca-file`: CAチェーンの保存パス
    - `--reloadcmd`: 証明書インストール後に自動的に実行するコマンド

#### 証明書自動更新の設定

acme.shはインストール時に自動的にcronジョブを登録し、証明書の有効期限を確認して更新します。

##### 更新の前提条件

- 初回発行履歴が必要です。
- `~/.acme.sh/<ドメイン>/`ディレクトリに証明書ファイルが存在する必要があります。

##### 自動更新設定

acme.shインストール時に自動的にcronジョブが登録されます。

**基本更新周期**: 満了60日前から自動更新を試行

**Cronジョブの確認**

```bash
crontab -l | grep acme.sh
```

一般的に以下のような形式で登録されます。

```
0 0 * * * "/home/user/.acme.sh"/acme.sh --cron --home "/home/user/.acme.sh" > /dev/null
```

##### 手動更新

必要に応じて次のコマンドで手動更新を実行できます。

```bash
# 全ての証明書更新試行
acme.sh --cron

# 特定の証明書強制更新
acme.sh --renew -d example.com --force
```

##### 更新周期の変更

acme.shは基本的に満了60日前から更新を試みます。この値を変更するには`~/.acme.sh/account.conf`ファイルを修正します。

```bash
# account.conf編集
vim ～/.acme.sh/account.conf
```

以下の行を追加または修正します。

```
Le_RenewalDays=30
```

このように設定すると、満了30日前から更新を試みます。

!!! danger "注意"
    - ED25519キータイプは現在acme.shでサポートしていない場合があります。証明書チェーンにED25519アルゴリズムを使用する証明書が含まれている場合、発行及び更新はできません。
    - 証明書チェーンは、最低3段階以上(Root → Intermediate → Leaf)で構成される必要があります。
    - `Le_RenewalDays`値は、ACMEサーバーのRate Limitポリシーを考慮して慎重に調整する必要があります。
    - Standaloneモードを使用する場合、更新時に80番ポートが使用可能である必要があるため、Webサーバーを一時中断するスクリプトが必要になる場合があります。

#### トラブルシューティング

##### 証明書発行失敗時

1. **ACME Directory URL確認**: `--server`オプションのURLが正しいか確認します。
2. **EAB認証情報確認**: アカウント登録が完了しているか確認します。登録されていない場合、`--register-account`コマンドでまずアカウントを登録します。
3. **ドメイン検証失敗**: 80番ポートが開いており、使用可能か確認します。既存のWebサーバーが80番ポートを使用中の場合、一時的に中断する必要があります。
4. **デバッグモード**: `--debug`オプションを追加して詳細ログを確認します。

```bash
acme.sh --issue \
  -d example.com \
  --server https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory \
  --keylength 2048 \
  --standalone \
  --debug
```

##### 証明書更新失敗時

1. **証明書情報確認**: 初回発行時に使用した設定が維持されているか確認します。
2. **Cronジョブ確認**: Cronジョブが正常に登録されているか確認します。
3. **ログ確認**: `~/.acme.sh/<ドメイン>/<ドメイン>.log`ファイルでエラーメッセージを確認します。
4. **手動更新テスト**: `acme.sh --renew -d example.com --force --debug`コマンドで手動更新を試行して問題を診断します。

## ACMEプロトコル情報

Private CAが提供するACME Directory URL(`/directory`)を通じて、ACMEクライアントは必要なすべてのエンドポイント情報を自動的に取得します。

ACMEプロトコルの全体の流れはクライアントによって自動的に処理されるため、ユーザーはDirectory URLのみ提供すれば済みます。

ACMEプロトコルの詳細については、[RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555)を参照してください。

## 参考資料

- [Certbot公式ドキュメント](https://certbot.eff.org/)
- [acme.sh公式ドキュメント](https://github.com/acmesh-official/acme.sh)
- [acme.sh DNS API一覧](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)
- [ACMEプロトコル仕様(RFC 8555)](https://datatracker.ietf.org/doc/html/rfc8555)
- [Let's Encrypt - Challenge Types](https://letsencrypt.org/docs/challenge-types/)
- [ACME証明書更新ガイド(cert-manager)](cert-manager-guide.md)
