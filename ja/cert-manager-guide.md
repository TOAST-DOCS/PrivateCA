# ACME証明書更新ガイド(cert-manager)
**Management > Private CA > ACME証明書更新ガイド(cert-manager)**

Private CAサービスは、ACME(automatic certificate management environment)プロトコルに対応し、証明書の自動発行及び更新を可能にします。cert-managerは、Kubernetes環境で証明書を自動的に管理するネイティブコントローラーであり、手動操作なしに証明書を発行し、定期的に更新できます。

本ガイドでは、Private CA ACMEサーバーを利用してKubernetesでcert-managerにより証明書を発行し、自動的に更新する方法を案内します。

!!! tip 「ポイント」
    - **ACME(automatic certificate management environment)**: 証明書の発行と更新を自動化する標準プロトコルです。
    - **cert-manager**: KubernetesでTLS証明書を自動的に管理するネイティブコントローラーです。
    - **Base証明書**: ACME自動更新時に参照するテンプレート役割の証明書です。
    - **CSR(certificate signing request)**: 証明書の発行をリクエストするための署名リクエストファイルです。
    - **EAB(external account binding)**: ACMEサーバーに認証するためのアカウントバインディング情報です。

!!! tip 「ポイント」
    一般サーバー環境でCertbotまたはacme.shを使用して証明書を管理するには、[ACME証明書更新ガイド(Certbot, acme.sh)](acme-guide.md)を参照してください。

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

## cert-managerを利用した証明書更新

Kubernetes環境では、cert-managerを使用して証明書を自動的に発行及び更新できます。

### cert-managerのインストール

Kubernetesクラスターにcert-managerをインストールします。

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.19.1/cert-manager.yaml
```

**インストール確認**

```bash
kubectl get pods -n cert-manager
```

### Ingress Controllerのインストール

HTTP-01 Challenge方式を使用するには、Ingress Controllerが必要です。

**ingress-nginxインストール**

```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

**インストール確認**

```bash
kubectl get pods -n ingress-nginx
```

### EAB Secretの作成

ACME認証のためのEAB(external account binding)情報をKubernetes Secretとして作成します。

```bash
kubectl create secret generic acme-eab-secret \
  --from-literal=secret='YOUR_ACME_TOKEN_HMAC_KEY' \
  --namespace default
```

!!! danger "注意"
    - `YOUR_ACME_TOKEN_HMAC_KEY`は、Private CAコンソール > ACME管理で発行されたACME HMACキーに置き換える必要があります。
    - EAB Secretは機密情報であるため、安全に管理する必要があります。
    - Secretが作成されたネームスペースとIssuerが配置されるネームスペースが同一である必要があります。

### Issuer設定

Issuerは、証明書の発行を受けるCAを定義するcert-managerリソースです。Namespace単位で動作するIssuerと、クラスター全体で使用可能なClusterIssuerの中から選択して使用できます。

HTTP-01 Challenge検証方式として、**Ingress**または**Gateway API**のいずれかを選択して使用できます。

#### 方法1: Ingressを利用した Issuer設定

Ingress Controllerを使用する場合のIssuer設定例です。

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: my-acme-issuer-example-com
  namespace: default
spec:
  acme:
    server: https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory

    # --- EAB設定追加開始 ---
    externalAccountBinding:
      keyID: "9998617a-2799-41dd-a75b-ff093f5472c7" # 例: "kid-1"、"my-account-id"など、サーバーから受け取ったID
      keySecretRef:
        name: acme-eab-secret # 上記で作成したSecret名
        key: secret           # Secret内部のKey名
    # --- EAB設定追加終了 ---

    privateKeySecretRef:
      name: my-acme-account-key-example-com # cert-managerが自動的に作成します。
    skipTLSVerify: true  # プライベート証明書サーバーであるため必須
    solvers:
    - http01:
        ingress:
          class: nginx
```
#### 方法2: Gateway APIを利用したIssuer設定
Kubernetes Gateway APIを使用する場合のIssuer設定です。
**事前準備**
1. **Gateway APIのインストール**
```bash
kubectl apply -f "https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml"
```

2. **cert-managerでGateway API機能を有効化**

**Helmでインストールした場合**

```bash
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set "extraArgs={--enable-gateway-api}"
```

**マニフェストでインストールした場合**

```bash
kubectl edit deployment cert-manager -n cert-manager
```

以下の内容を追加します。

```yaml
spec:
  template:
    spec:
      containers:
      - name: cert-manager
        args:
        - --enable-gateway-api
```
3. **Traefikのインストール(Gateway Controllerの例)**
```bash
# Helmリポジトリ追加
helm repo add traefik https://traefik.github.io/charts

# Traefikインストール
helm install traefik traefik/traefik -n traefik --create-namespace -f traefik-values.yaml
```

**traefik-values.yaml**

```yaml
experimental:
  kubernetesGateway:
    enabled: true

providers:
  kubernetesGateway:
    enabled: true

# GatewayClass自動作成
gatewayClass:
  enabled: true
  name: traefik

# Gateway自動作成無効化(手動で作成すること)
gateway:
  enabled: false

# 内部ポートを1024以上に設定
ports:
  web:
    port: 8000          # 内部ポート
    exposedPort: 80     # 外部ポート
  websecure:
    port: 8443          # 内部ポート
    exposedPort: 443    # 外部ポート

service:
  type: LoadBalancer

ingressRoute:
  dashboard:
    enabled: true

logs:
  general:
    level: INFO
```
4. **Gatewayリソースの作成**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: traefik
  namespace: default
spec:
  gatewayClassName: traefik
  listeners:
  # HTTP Listener(80ポート)
  - name: http
    protocol: HTTP
    port: 8000  # Traefikのweb entryPointと一致
    allowedRoutes:
      namespaces:
        from: All
  # HTTPS Listener(443ポート)
  - name: https
    protocol: HTTPS
    port: 8443  # Traefikのwebsecure entryPointと一致
    allowedRoutes:
      namespaces:
        from: All
    tls:
      mode: Terminate
      certificateRefs:
      - kind: Secret
        name: test-server-tls-example-com  # TLS Secret名
        namespace: default
```
Gatewayを適用します。
```bash
kubectl apply -f gateway.yml
```

**Issuer設定(Gateway API使用)**

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: my-acme-issuer-example-com
  namespace: default
spec:
  acme:
    server: https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory

    # --- EAB設定追加開始 ---
    externalAccountBinding:
      keyID: "9998617a-2799-41dd-a75b-ff093f5472c7"
      keySecretRef:
        name: acme-eab-secret
        key: secret
    # --- EAB設定追加終了 ---

    privateKeySecretRef:
      name: my-acme-account-key-example-com
    skipTLSVerify: true
    solvers:
    - http01:
        gatewayHTTPRoute:
          parentRefs:
            - name: traefik
              namespace: default
              kind: Gateway
```
!!! tip 「ポイント」
    Gateway API方式を使用すると、cert-managerが自動的にHTTPRoute、Service、Podを作成してChallengeを処理します。Gatewayリソースは事前に作成されている必要があります。
#### 主なフィールドの説明
| フィールド | 説明 | 必須 |
|------|------|------|
| `spec.acme.server` | ACME Directory URL。Private CA Base証明書IDを含みます。 | O |
| `spec.acme.externalAccountBinding.keyID` | ACMEトークンID。Private CAコンソールで発行された値を入力します。 | O |
| `spec.acme.externalAccountBinding.keySecretRef.name` | EAB HMACキーが保存されたSecret名。 | O |
| `spec.acme.externalAccountBinding.keySecretRef.key` | Secret内部のKey名。 | O |
| `spec.acme.privateKeySecretRef.name` | ACMEアカウントの秘密鍵を保存するSecret名。cert-managerが自動的に作成します。 | O |
| `spec.acme.skipTLSVerify` | TLS証明書の検証をスキップするかどうか。プライベート証明書環境では`true`に設定します。 | X |
| `spec.acme.solvers` | ACME Challenge検証方式。`http01`、`dns01`の中から選択します。 | O |
| `spec.acme.solvers.http01.ingress.class` | Ingress方式: Ingress Controllerクラス名(例: `nginx`)。 | X |
| `spec.acme.solvers.http01.gatewayHTTPRoute.parentRefs` | Gateway API方式: 参照するGatewayリソース情報。 | X |

#### Issuerの適用と状態確認

Issuerリソースを適用します。

```bash
kubectl apply -f my-acme-issuer-example-com.yml
```

Issuerの状態を確認します。

```bash
kubectl get issuer -n default
kubectl describe issuers.cert-manager.io my-acme-issuer-example-com -n default
```

`Ready`ステータスが`True`と表示されれば、正常に登録されています。

!!! tip 「ポイント」
    - Ingress方式: Ingress Controllerがインストールされている必要があります。
    - Gateway API方式: Gateway API CRDのインストール、cert-managerでのGateway API機能有効化、Gateway Controller(例: Traefik)のインストール、Gatewayリソースの作成が必要です。
    - `dns01` solverはDNSプロバイダー設定が必要です。
    - `skipTLSVerify: true`オプションは、Private CAサーバーがプライベート証明書を使用する場合に必須です。

#### HTTP-01 ChallengeのためのhostAliases設定(先行作業)

Certificateを作成する前に、HTTP-01 Challengeが成功するように設定する必要があります。

HTTP-01 Challengeは、cert-managerがドメインの所有権を検証するために、該当ドメインへHTTPリクエストを送信します。テスト環境でドメインが実際のDNSに登録されていない場合、cert-manager DeploymentにhostAliasesを設定し、ドメインをKubernetesクラスターのIPにマッピングする必要があります。

```bash
kubectl edit deployment cert-manager -n cert-manager
```

以下の内容を`spec.template.spec`の下に追加します。

```yaml
spec:
  template:
    spec:
      hostAliases:
      - ip: "114.110.145.74"  # Kubernetesクラスターの外部IP(例: NKS フローティングIP、LoadBalancer IP)
        hostnames:
        - "example.com"       # Certificateに設定した全てのドメイン
        - "sub.example.com"
```

この設定は、cert-manager Podがドメインへリクエストを送信する際、指定したIPに接続するようにします。

!!! danger "注意"
    - IPアドレスは、KubernetesクラスターのIngress ControllerまたはGatewayのLoadBalancer外部IPに設定する必要があります。
    - Certificateに設定した**全てのドメイン**(`commonName`及び`dnsNames`)を`hostnames`に追加する必要があります。
    - 本番環境では、実際のDNSレコードを設定することが推奨されます。

!!! tip 「ポイント」
    Gateway APIを使用する場合、cert-managerは自動的に以下のリソースを作成してChallengeを処理します。

    1. **HTTPRoute**: 各ドメインごとにChallengeパス(`/.well-known/acme/v2.0-challenge/`)を処理するHTTPRouteを作成します。
    2. **Service**: Challengeを処理するServiceを作成します。
    3. **Pod**: Challengeに応答するPodを作成します。

    これらのリソースはChallenge完了後、自動的に削除されます。

### Certificateリソースの作成

Certificateリソースは、発行する証明書の属性を定義します。

#### Certificate設定例

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: test-server-cert-example-com
  namespace: default
spec:
  secretName: test-server-tls-example-com # 証明書が保存されるSecret
  renewBefore: 360h # 15d
  commonName: example.com
  dnsNames:
   - example.com
   - sub.example.com
  # Issuer references are always required.
  issuerRef:
    name: my-acme-issuer-example-com
    # We can reference ClusterIssuers by changing the kind here.
    # The default value is Issuer(i.e. a locally namespaced Issuer)
    kind: Issuer
```

#### 主なフィールドの説明

| フィールド | 説明 | 必須 |
|------|------|------|
| `spec.secretName` | 発行された証明書と秘密鍵を保存するSecret名。 | O |
| `spec.renewBefore` | 証明書満了前の更新開始時間。例: `360h`(15日)。 | X |
| `spec.commonName` | 証明書のCommon Name(CN)。Base証明書のCNと一致する必要があります。 | O |
| `spec.dnsNames` | 証明書のSubject Alternative Names(SAN)。Base証明書のSANと一致する必要があります。 | X |
| `spec.issuerRef.name` | 使用するIssuerまたはClusterIssuer名。 | O |
| `spec.issuerRef.kind` | Issuerタイプ。`Issuer`または`ClusterIssuer`。 | X |

#### Certificateの適用と状態確認

Certificateリソースを適用します。

```bash
kubectl apply -f test-server-cert-example-com.yml
```

Certificateの状態を確認します。

```bash
kubectl get certificate -n default
kubectl describe certificate test-server-cert-example-com -n default
```

証明書の発行に成功すると、`Ready`ステータスが`True`と表示されます。

**証明書発行の進行状況詳細確認**

```bash
# CertificateRequest確認(注文作成)
kubectl get certificaterequest -n default
# Order確認(注文処理)
kubectl get order -n default
# Challenge確認(チャレンジ検証)
kubectl get challenge -n default
```

!!! danger "注意"
    - `commonName`と`dnsNames`に指定したドメインは、Base証明書に設定されたCN及びSANと正確に一致する必要があります。
    - Base証明書にないドメインを追加すると、証明書の発行に失敗します。
    - 証明書発行前に、コンソールでBase証明書のCNとSAN情報を確認し、正しいドメインを指定しているか必ず検証してください。

### 発行された証明書の確認

証明書が正常に発行されると、指定したSecretに保存されます。

#### Secret確認

```bash
kubectl get secret test-server-tls-example-com -n default
kubectl describe secret test-server-tls-example-com -n default
```

#### 証明書内容の確認

**証明書詳細情報の確認**

```bash
kubectl get secret test-server-tls-example-com -n default -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -text
```

**証明書有効期限の確認**

```bash
kubectl get secret test-server-tls-example-com -n default -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -dates
```

#### Secret構造

発行された証明書Secretは以下のような構造を持ちます。

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded certificate>
  tls.key: <base64-encoded private key>
  ca.crt: <base64-encoded CA certificate chain>
```

### 証明書の自動更新

cert-managerは、証明書の有効期限が近づくと自動的に更新を実行します。

#### 自動更新の動作方式

- cert-managerは、定期的にCertificateリソースの有効期限を確認します。
- `renewBefore`フィールドに設定された時間分だけ有効期限が残っている場合、自動的に更新を開始します。
- 更新された証明書は、同一のSecretに自動的にアップデートされます。

#### 更新周期の設定

Certificateリソースの`renewBefore`フィールドを修正して、更新開始時点を調整できます。

```yaml
spec:
  renewBefore: 720h  # 30日前に更新開始
```

#### 手動更新

必要に応じてCertificateリソースを手動で更新できます。

**方法1: cmctlコマンドの使用**

```bash
# cmctlインストール(macOS)
brew install cmctl
# 証明書更新
cmctl renew test-server-cert-example-com -n default
```

**方法2: kubectl annotationの使用**

```bash
# Certificate annotationを利用した手動更新トリガー
kubectl annotate certificate test-server-cert-example-com -n default \
  cert-manager.io/issue-temporary-certificate="true" --overwrite
```

**方法3: Certificateリソースの再作成**

```bash
kubectl delete certificate test-server-cert-example-com -n default
kubectl apply -f test-server-cert-example-com.yml
```

!!! tip 「ポイント」
    - cert-managerは、基本的に満了30日前から更新を試みます。
    - `renewBefore`値を短く設定しすぎると、証明書が期限切れになる危険があるため注意が必要です。
    - 更新失敗時、cert-managerは自動的に再試行します。

#### 更新テスト方法

自動更新が正しく動作するかテストするには、以下の方法を使用できます。

**方法1:自動更新時間まで待機**

Base証明書発行時にTTLを短く設定し、Certificateの`renewBefore`値も短く設定することで、素早く更新プロセスを確認できます。

設定例
- Base証明書TTL: 2日
- Certificate `renewBefore`: 24h(満了24時間前に更新開始)

```yaml
spec:
  renewBefore: 24h  # 満了24時間前に更新開始
```

**方法2: 手動更新で即時テスト**

上記の手動更新方法(cmctlまたはkubectl annotation)を使用して、即座に更新プロセスをテストできます。

```bash
# cmctlを利用した更新
cmctl renew test-server-cert-example-com -n default
```

### アプリケーションでの証明書使用

発行された証明書はKubernetes Secretとして保存されるため、様々な方法でアプリケーションにマウントできます。

#### Ingressでの使用

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  namespace: default
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - example.com
    - sub.example.com
    secretName: test-server-tls-example-com  # Certificateで作成したSecret
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

#### PodでVolumeとしてマウント

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  namespace: default
spec:
  containers:
  - name: app
    image: my-app:latest
    volumeMounts:
    - name: tls-certs
      mountPath: /etc/tls
      readOnly: true
  volumes:
  - name: tls-certs
    secret:
      secretName: test-server-tls-example-com
```

マウントされた証明書は以下のパスで使用できます。

- `/etc/tls/tls.crt`: 証明書
- `/etc/tls/tls.key`: 秘密鍵
- `/etc/tls/ca.crt`: CA証明書チェーン

### トラブルシューティング

#### 証明書発行失敗時

1. **Issuer状態確認**

```bash
kubectl describe issuer my-acme-issuer-example-com -n default
```

- `Ready`ステータスが`True`か確認します。
- ACMEアカウント登録が成功したか確認します。

2. **Certificate状態確認**

```bash
kubectl describe certificate test-server-cert-example-com -n default
```

- Eventsセクションでエラーメッセージを確認します。

3. **CertificateRequest確認**

```bash
kubectl get certificaterequest -n default
kubectl describe certificaterequest <request-name> -n default
```

4. **Challenge確認**

```bash
kubectl get challenge -n default
kubectl describe challenge <challenge-name> -n default
```

- HTTP-01 Challengeの場合、Ingressが正しく設定されているか確認します。
- Challenge URLにアクセス可能か確認します。

#### 一般的なエラーと解決方法

| エラーメッセージ | 原因 | 解決方法 |
|-----------|------|----------|
| `Failed to register ACME account` | EAB情報が間違っています。 | `keyID`とSecret値を確認します。 |
| `challenge failed` | Challenge検証に失敗しました。 | Ingress設定及びネットワーク接続状況を確認します。 |
| `domain not allowed` | Base証明書にないドメインをリクエストしました。 | Certificateの`commonName`と`dnsNames`をBase証明書と一致させます。 |
| `secret not found` | EAB Secretが見つかりません。 | Secretが正しいネームスペースに作成されたか確認します。 |
| `x509: certificate signed by unknown authority` | TLS検証失敗。 | Issuerに`skipTLSVerify: true`を設定します。 |

#### ログ確認

cert-managerの詳細ログを確認して問題を診断できます。

```bash
kubectl logs -n cert-manager deployment/cert-manager -f
```

!!! danger "注意"
    - ED25519キータイプは、cert-managerの一部のバージョンでサポートされない場合があります。証明書チェーンにED25519アルゴリズムを使用する証明書が含まれている場合、発行および更新ができない可能性があります。
    - 証明書チェーンは、最低3段階以上(Root → Intermediate → Leaf)で構成される必要があります。
    - `renewBefore`値は、ACMEサーバーのRate Limitポリシーを考慮して慎重に調整する必要があります。

## ACMEプロトコル情報

Private CAが提供するACME Directory URL(`/directory`)を通じて、ACMEクライアントは必要なすべてのエンドポイント情報を自動的に取得します。

ACMEプロトコルの全体の流れはクライアントによって自動的に処理されるため、ユーザーはDirectory URLのみ提供すれば済みます。

ACMEプロトコルの詳細については、[RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555)を参照してください。

## 参考資料

- [cert-manager公式ドキュメント](https://cert-manager.io/docs/)
- [cert-manager ACME設定ガイド](https://cert-manager.io/docs/configuration/acme/)
- [ACMEプロトコル仕様(RFC 8555)](https://datatracker.ietf.org/doc/html/rfc8555)
- [Let's Encrypt - Challenge Types](https://letsencrypt.org/docs/challenge-types/)
- [Kubernetes Ingress TLS設定](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)
- [ACME証明書更新ガイド(Certbot, acme.sh)](acme-guide.md)
