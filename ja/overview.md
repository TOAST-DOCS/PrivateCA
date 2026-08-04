<!-- pre-align:aligned sig=5eca1c1afe25 -->

# 概要
**Management > Private CA > 概要**

Private CAは組織内部で使用する証明書を直接発行し管理できるサービスです。公認認証局を経由せずに安全に証明書を発行し、内部システム、APIサーバー、IoTデバイスなどに適用できます。

<a id="main-features"></a>
## 主な特徴 { #main-features }

<a id="run-your-own-certificate-authority-ca"></a>
### 独自認証局(CA)運営 { #run-your-own-certificate-authority-ca }
- Root CAとIntermediate CAを作成して証明書階層構造を構成できます。
- 組織のセキュリティポリシーに合わせて認証局を管理及び運営できます。
- 外部認証局に依存せず、独立して証明書を発行できます。

<a id="automatically-issue-and-renew-certificates"></a>
### 証明書自動発行及び更新 { #automatically-issue-and-renew-certificates }
- 証明書テンプレートを使用して、同じ設定の証明書を迅速かつ一貫性を持って発行できます。
- ACME(automatic certificate management environment)プロトコルをサポートし、証明書の発行及び更新を自動化できます。
- Certbotなどの標準ACMEクライアントと互換性があります。

<a id="manage-certificate-retirement"></a>
### 証明書失効管理 { #manage-certificate-retirement }
- CRL(certificate revocation list)を通じて失効した証明書リストを定期的に提供します。
- OCSP(online certificate status protocol)を通じて個別証明書の失効状態をリクエスト時点の状態で素早く確認できます。
- 証明書失効履歴を追跡し、監査できます。

<a id="api-support"></a>
### APIサポート { #api-support }
- RESTful APIを通じて証明書をプログラムで管理できます。
- 証明書ダウンロード、CRL照会、OCSPレスポンスなどの機能をAPIで提供します。
- 自動化システムと簡単に統合できます。

<a id="configure-a-service"></a>
## サービス構成 { #configure-a-service }

Private CAサービスは次のようなコンポーネントで構成されています。

![Private CAサービス構造](https://static.toastoven.net/prod_privateca/2025-12-23_ko/NHN%20Cloud_PrivateCA_overview_ja_900.png)

<a id="repository"></a>
### リポジトリ { #repository }
- Private CAを管理する基本単位です。
- 発行者、証明書テンプレート、証明書、ACMEトークンなど、全てのリソースは特定のリポジトリに属します。
- CRL及びOCSP設定をリポジトリ単位で管理します。

<a id="issuer"></a>
### 発行者 { #issuer }
- 証明書に署名し発行する認証局です。
- **Root CA**：最上位認証局で、自己署名された証明書です。全ての信頼の起点となります。
- **Intermediate CA**：Root CAによって署名された中間認証局です。実際のサーバー証明書発行に使用されます。

<a id="certificate-template"></a>
### 証明書テンプレート { #certificate-template }
- 証明書を迅速かつ一貫性を持って発行するための設定の集まりです。
- 同じ設定で複数の証明書を簡単に発行でき、次の2つの設定で構成されます。
    - **制限設定**：有効期間、SANオプションなど、証明書発行時の制限条件を定義します。
    - **共通反映設定**：鍵アルゴリズム、鍵使用用途、拡張鍵使用用途、主体情報など、証明書に共通して適用する設定を定義します。

<a id="certificate"></a>
### 証明書 { #certificate }
- 発行者が署名した実際に使用可能な証明書です。
- サーバー認証、クライアント認証、コード署名など、様々な用途に使用できます。
- PEM形式でダウンロードしてシステムに適用できます。

<a id="acme-token"></a>
### ACMEトークン { #acme-token }
- ACMEプロトコルを通じた自動証明書発行に使用される認証情報です。
- CertbotなどのACMEクライアントと連携して証明書を自動的に発行及び更新できます。
- トークンIDとHMACキーを使用してACMEサーバーに認証します。

<a id="certificate-issuance-workflow"></a>
## 証明書発行フロー { #certificate-issuance-workflow }

Private CAで証明書を発行する基本フローは次のとおりです。

1. **リポジトリ作成**：証明書を管理するスペースを作成します。
2. **発行者作成**：Root CAまたはIntermediate CAを作成します。
3. **証明書テンプレート作成**：証明書発行に使用するテンプレートを作成します。
4. **証明書発行**：テンプレートを使用して実際の証明書を発行します。

発行された証明書はPEM形式でダウンロードしてWebサーバー、APIサーバー、アプリケーションなどに適用できます。

<a id="use-cases"></a>
## 活用事例 { #use-cases }

<a id="enhance-internal-system-security"></a>
### 内部システムセキュリティ強化 { #enhance-internal-system-security }
- 組織内部のWebサーバー、APIサーバー、データベースなどにTLS/SSL証明書を発行して通信を暗号化できます。
- 内部インフラに公認証明書を使用する必要がなく、費用を削減できます。

<a id="cross-microservice-authentication"></a>
### マイクロサービス間認証 { #cross-microservice-authentication }
- マイクロサービスアーキテクチャでサービス間相互認証(mTLS)に使用する証明書を発行できます。
- サービスメッシュ(Service Mesh)環境で安全な通信を実装できます。

<a id="authenticate-iot-devices"></a>
### IoTデバイス認証 { #authenticate-iot-devices }
- IoTデバイスに固有の証明書を発行してデバイス認証及び通信セキュリティを強化できます。
- 大量のデバイスに自動的に証明書を配布及び管理できます。

<a id="development-and-test-environments"></a>
### 開発及びテスト環境 { #development-and-test-environments }
- 開発及びテスト環境で実際の運用環境と同じ証明書構造を再現できます。
- 安全なテスト環境を構築してセキュリティの脆弱性を事前に発見できます。

<a id="code-signing-and-document-signing"></a>
### コード署名及びドキュメント署名 { #code-signing-and-document-signing }
- ソフトウェア配布時にコード署名証明書を発行してソフトウェアの整合性を保証できます。
- 電子文書にデジタル署名を適用して文書の真偽を確認できます。

<a id="getting-started"></a>
## 始める { #getting-started }

Private CAを初めて使用する場合は、次のガイドを参考にしてください。

- [コンソール利用ガイド](./console-guide.md)：Private CAコンソールでリポジトリ、発行者、証明書テンプレート、証明書を作成及び管理する方法を案内します。
- [ACME証明書更新ガイド(Certbot, acme.sh)](./client-guide.md)：Certbotまたはacme.shを使用して証明書を自動的に発行及び更新する方法を案内します。
- [API v2.0ガイド](./api-guide-v2.0.md)：APIを通じて証明書をダウンロードし、CRL、OCSPを照会する方法を案内します。

!!! tip "ポイント"
    - Private CAは組織内部用証明書の発行に最適化されています。公認証明書が必要な場合は、公認認証局を利用する必要があります。
    - 発行された証明書を使用するには、クライアントシステムにCAチェーンを信頼できる証明書として登録する必要があります。
    - ACMEプロトコルを使用すると、証明書の発行及び更新を完全に自動化でき、運用負担を大幅に軽減できます。

!!! danger "注意"
    - 証明書の失効は元に戻せない作業です。慎重に決定する必要があります。
    - CRL及びOCSPを有効化して、失効した証明書をクライアントが確認できるように設定する必要があります。
