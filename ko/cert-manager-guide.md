# ACME 인증서 갱신 가이드(cert-manager)
**Management > Private CA > ACME 인증서 갱신 가이드(cert-manager)**

Private CA 서비스는 ACME(automatic certificate management environment) 프로토콜을 지원하여 인증서의 자동 발급 및 갱신을 가능하게 합니다. cert-manager는 Kubernetes 환경에서 인증서를 자동으로 관리하는 네이티브 컨트롤러로, 수동 개입 없이 인증서를 발급 받고 주기적으로 갱신할 수 있습니다.

본 가이드에서는 Private CA ACME 서버를 이용하여 Kubernetes에서 cert-manager로 인증서를 발급하고 자동으로 갱신하는 방법을 안내합니다.

!!! tip "알아두기"
    - **ACME(automatic certificate management environment)**: 인증서 발급과 갱신을 자동화하는 표준 프로토콜입니다.
    - **cert-manager**: Kubernetes에서 TLS 인증서를 자동으로 관리하는 네이티브 컨트롤러입니다.
    - **Base 인증서**: ACME 자동 갱신 시 참조하는 템플릿 역할의 인증서입니다.
    - **CSR(certificate signing request)**: 인증서 발급을 요청하기 위한 서명 요청 파일입니다.
    - **EAB(external account binding)**: ACME 서버에 인증하기 위한 계정 바인딩 정보입니다.

!!! tip "알아두기"
    일반 서버 환경에서 Certbot 또는 acme.sh를 사용하여 인증서를 관리하려면 [ACME 인증서 갱신 가이드(Certbot, acme.sh)](acme-guide.md)를 참고하세요.

## 사전 준비하기

ACME를 통한 인증서 발급을 시작하기 전에 다음 사항을 준비해야 합니다.

### 1. Base 인증서 발급

Base 인증서는 ACME 서버가 자동 갱신 시 참조하는 "템플릿" 역할을 합니다.

- Base 인증서에 설정된 도메인(CN, SAN)과 동일한 도메인의 인증서만 ACME를 통해 갱신할 수 있습니다.
- Base 인증서는 콘솔에서 일반적인 인증서 발급 절차로 생성합니다.
- Base 인증서를 발급 받은 후, 해당 인증서의 ID를 ACME Directory URL에 사용합니다.

### 2. ACME 서버 정보 확인

Private CA 콘솔에서 다음 정보를 확인합니다.

- **ACME Directory URL**: `https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory`
- **ACME 토큰 ID**: 콘솔에서 발급한 ACME 토큰 ID(**YOUR_ACME_TOKEN_ID**)
- **ACME HMAC 키**: 콘솔에서 발급한 ACME 토큰 HMAC 키(**YOUR_ACME_TOKEN_HMAC_KEY**)

## cert-manager를 이용한 인증서 갱신

Kubernetes 환경에서는 cert-manager를 사용하여 인증서를 자동으로 발급하고 갱신할 수 있습니다.

### cert-manager 설치

Kubernetes 클러스터에 cert-manager를 설치합니다.

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.19.1/cert-manager.yaml
```

**설치 확인**

```bash
kubectl get pods -n cert-manager
```

### Ingress Controller 설치

HTTP-01 Challenge 방식을 사용하려면 Ingress Controller가 필요합니다.

**ingress-nginx 설치**

```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

**설치 확인**

```bash
kubectl get pods -n ingress-nginx
```

### EAB Secret 생성

ACME 인증을 위한 EAB(external account binding) 정보를 Kubernetes Secret으로 생성합니다.

```bash
kubectl create secret generic acme-eab-secret \
  --from-literal=secret='YOUR_ACME_TOKEN_HMAC_KEY' \
  --namespace default
```

!!! danger "주의"
    - `YOUR_ACME_TOKEN_HMAC_KEY`는 Private CA 콘솔 > ACME 관리에서 발급 받은 ACME HMAC 키로 교체해야 합니다.
    - EAB Secret은 민감한 정보이므로 안전하게 관리해야 합니다.
    - Secret이 생성된 네임스페이스와 Issuer가 위치할 네임스페이스가 동일해야 합니다.

### Issuer 설정

Issuer는 인증서를 발급 받을 CA를 정의하는 cert-manager 리소스입니다. Namespace 단위로 동작하는 Issuer와 클러스터 전체에서 사용 가능한 ClusterIssuer 중 선택하여 사용할 수 있습니다.

HTTP-01 Challenge 검증 방식으로 **Ingress** 또는 **Gateway API** 중 하나를 선택하여 사용할 수 있습니다.

#### 방법 1: Ingress를 이용한 Issuer 설정

Ingress Controller를 사용하는 경우의 Issuer 설정 예시입니다.

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: my-acme-issuer-example-com
  namespace: default
spec:
  acme:
    server: https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory

    # --- EAB 설정 추가 시작 ---
    externalAccountBinding:
      keyID: "9998617a-2799-41dd-a75b-ff093f5472c7" # 예: "kid-1", "my-account-id" 등 서버에서 받은 ID
      keySecretRef:
        name: acme-eab-secret # 위에서 만든 Secret 이름
        key: secret           # Secret 내부의 Key 이름
    # --- EAB 설정 추가 끝 ---

    privateKeySecretRef:
      name: my-acme-account-key-example-com # cert-manager가 자동으로 생성해줌.
    skipTLSVerify: true  # 사설 인증서 서버이므로 필수
    solvers:
    - http01:
        ingress:
          class: nginx
```

#### 방법 2: Gateway API를 이용한 Issuer 설정

Kubernetes Gateway API를 사용하는 경우의 Issuer 설정입니다.

**사전 준비**

1. **Gateway API 설치**

```bash
kubectl apply -f "https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml"
```

2. **cert-manager에 Gateway API 기능 활성화**

**Helm으로 설치한 경우**

```bash
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set "extraArgs={--enable-gateway-api}"
```

**Manifest로 설치한 경우**

```bash
kubectl edit deployment cert-manager -n cert-manager
```

다음 내용을 추가합니다.

```yaml
spec:
  template:
    spec:
      containers:
      - name: cert-manager
        args:
        - --enable-gateway-api
```

3. **Traefik 설치(Gateway Controller 예시)**

```bash
# Helm 저장소 추가
helm repo add traefik https://traefik.github.io/charts

# Traefik 설치
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

# GatewayClass 자동 생성
gatewayClass:
  enabled: true
  name: traefik

# Gateway 자동 생성 비활성화(수동으로 만들 것)
gateway:
  enabled: false

# 내부 포트를 1024 이상으로 설정
ports:
  web:
    port: 8000          # 내부 포트
    exposedPort: 80     # 외부 포트
  websecure:
    port: 8443          # 내부 포트
    exposedPort: 443    # 외부 포트

service:
  type: LoadBalancer

ingressRoute:
  dashboard:
    enabled: true

logs:
  general:
    level: INFO
```

4. **Gateway 리소스 생성**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: traefik
  namespace: default
spec:
  gatewayClassName: traefik
  listeners:
  # HTTP Listener(80 포트)
  - name: http
    protocol: HTTP
    port: 8000  # Traefik의 web entryPoint와 일치
    allowedRoutes:
      namespaces:
        from: All
  # HTTPS Listener(443 포트)
  - name: https
    protocol: HTTPS
    port: 8443  # Traefik의 websecure entryPoint와 일치
    allowedRoutes:
      namespaces:
        from: All
    tls:
      mode: Terminate
      certificateRefs:
      - kind: Secret
        name: test-server-tls-example-com  # TLS Secret 이름
        namespace: default
```

Gateway를 적용합니다.

```bash
kubectl apply -f gateway.yml
```

**Issuer 설정(Gateway API 사용)**

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: my-acme-issuer-example-com
  namespace: default
spec:
  acme:
    server: https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory

    # --- EAB 설정 추가 시작 ---
    externalAccountBinding:
      keyID: "9998617a-2799-41dd-a75b-ff093f5472c7"
      keySecretRef:
        name: acme-eab-secret
        key: secret
    # --- EAB 설정 추가 끝 ---

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

!!! tip "알아두기"
    Gateway API 방식을 사용하면 cert-manager가 자동으로 HTTPRoute, Service, Pod를 생성하여 Challenge를 처리합니다. Gateway 리소스는 미리 생성되어 있어야 합니다.

#### 주요 필드 설명

| 필드 | 설명 | 필수 |
|------|------|------|
| `spec.acme.server` | ACME Directory URL. Private CA Base 인증서 ID를 포함합니다. | O |
| `spec.acme.externalAccountBinding.keyID` | ACME 토큰 ID. Private CA 콘솔에서 발급 받은 값을 입력합니다. | O |
| `spec.acme.externalAccountBinding.keySecretRef.name` | EAB HMAC 키가 저장된 Secret 이름. | O |
| `spec.acme.externalAccountBinding.keySecretRef.key` | Secret 내부의 Key 이름. | O |
| `spec.acme.privateKeySecretRef.name` | ACME 계정 개인 키를 저장할 Secret 이름. cert-manager가 자동으로 생성합니다. | O |
| `spec.acme.skipTLSVerify` | TLS 인증서 검증 스킵 여부. 사설 인증서 환경에서는 `true`로 설정합니다. | X |
| `spec.acme.solvers` | ACME Challenge 검증 방식. `http01`, `dns01` 중 선택합니다. | O |
| `spec.acme.solvers.http01.ingress.class` | Ingress 방식: Ingress Controller 클래스 이름(예: `nginx`). | X |
| `spec.acme.solvers.http01.gatewayHTTPRoute.parentRefs` | Gateway API 방식: 참조할 Gateway 리소스 정보. | X |

#### Issuer 적용 및 상태 확인

Issuer 리소스를 적용합니다.

```bash
kubectl apply -f my-acme-issuer-example-com.yml
```

Issuer 상태를 확인합니다.

```bash
kubectl get issuer -n default
kubectl describe issuers.cert-manager.io my-acme-issuer-example-com -n default
```

`Ready` 상태가 `True`로 표시되면 정상적으로 등록된 것입니다.

!!! tip "알아두기"
    - Ingress 방식: Ingress Controller가 설치되어 있어야 합니다.
    - Gateway API 방식: Gateway API CRD 설치, cert-manager에 Gateway API 기능 활성화, Gateway Controller(예: Traefik) 설치, Gateway 리소스 생성이 필요합니다.
    - `dns01` solver는 DNS 프로바이더 설정이 필요합니다.
    - `skipTLSVerify: true` 옵션은 Private CA 서버가 사설 인증서를 사용하는 경우 필수입니다.

#### HTTP-01 Challenge를 위한 hostAliases 설정(선행 작업)

Certificate를 생성하기 전에 HTTP-01 Challenge가 성공할 수 있도록 설정해야 합니다.

HTTP-01 Challenge는 cert-manager가 도메인의 소유권을 검증하기 위해 해당 도메인으로 HTTP 요청을 보냅니다. 테스트 환경에서 도메인이 실제 DNS에 등록되지 않은 경우, cert-manager Deployment에 hostAliases를 설정하여 도메인을 Kubernetes 클러스터의 IP로 매핑해야 합니다.

```bash
kubectl edit deployment cert-manager -n cert-manager
```

다음 내용을 `spec.template.spec` 아래에 추가합니다.

```yaml
spec:
  template:
    spec:
      hostAliases:
      - ip: "114.110.145.74"  # Kubernetes 클러스터의 외부 IP(예: NKS Floating IP, LoadBalancer IP)
        hostnames:
        - "example.com"       # Certificate에 설정한 모든 도메인
        - "sub.example.com"
```

이 설정은 cert-manager Pod가 도메인으로 요청을 보낼 때 지정한 IP로 연결하도록 합니다.

!!! danger "주의"
    - IP 주소는 Kubernetes 클러스터의 Ingress Controller 또는 Gateway의 LoadBalancer 외부 IP로 설정해야 합니다.
    - Certificate에 설정한 **모든 도메인**(`commonName` 및 `dnsNames`)을 `hostnames`에 추가해야 합니다.
    - 프로덕션 환경에서는 실제 DNS 레코드를 설정하는 것이 권장됩니다.

!!! tip "알아두기"
    Gateway API를 사용하는 경우, cert-manager는 자동으로 다음 리소스를 생성하여 Challenge를 처리합니다.

    1. **HTTPRoute**: 각 도메인마다 Challenge 경로(`/.well-known/acme/v2.0-challenge/`)를 처리할 HTTPRoute를 생성합니다.
    2. **Service**: Challenge를 처리할 Service를 생성합니다.
    3. **Pod**: Challenge를 응답할 Pod를 생성합니다.

    이러한 리소스는 Challenge 완료 후 자동으로 삭제됩니다.

### Certificate 리소스 생성

Certificate 리소스는 발급 받을 인증서의 속성을 정의합니다.

#### Certificate 설정 예시

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: test-server-cert-example-com
  namespace: default
spec:
  secretName: test-server-tls-example-com # 인증서가 저장될 시크릿
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

#### 주요 필드 설명

| 필드 | 설명 | 필수 |
|------|------|------|
| `spec.secretName` | 발급된 인증서와 개인 키를 저장할 Secret 이름. | O |
| `spec.renewBefore` | 인증서 만료 전 갱신 시작 시간. 예: `360h`(15일). | X |
| `spec.commonName` | 인증서의 Common Name(CN). Base 인증서의 CN과 일치해야 합니다. | O |
| `spec.dnsNames` | 인증서의 Subject Alternative Names(SAN). Base 인증서의 SAN과 일치해야 합니다. | X |
| `spec.issuerRef.name` | 사용할 Issuer 또는 ClusterIssuer 이름. | O |
| `spec.issuerRef.kind` | Issuer 유형. `Issuer` 또는 `ClusterIssuer`. | X |

#### Certificate 적용 및 상태 확인

Certificate 리소스를 적용합니다.

```bash
kubectl apply -f test-server-cert-example-com.yml
```

Certificate 상태를 확인합니다.

```bash
kubectl get certificate -n default
kubectl describe certificate test-server-cert-example-com -n default
```

인증서 발급이 성공하면 `Ready` 상태가 `True`로 표시됩니다.

**인증서 발급 진행 상황 상세 확인**

```bash
# CertificateRequest 확인(주문 생성)
kubectl get certificaterequest -n default

# Order 확인(주문 처리)
kubectl get order -n default

# Challenge 확인(챌린지 검증)
kubectl get challenge -n default
```

!!! danger "주의"
    - `commonName`과 `dnsNames`에 지정한 도메인은 Base 인증서에 설정된 CN과 SAN과 정확히 일치해야 합니다.
    - Base 인증서에 없는 도메인을 추가하면 인증서 발급이 실패합니다.
    - 인증서 발급 전 콘솔에서 Base 인증서의 CN과 SAN 정보를 확인하여 올바른 도메인을 지정했는지 반드시 검증하세요.

### 발급된 인증서 확인

인증서가 성공적으로 발급되면 지정한 Secret에 저장됩니다.

#### Secret 확인

```bash
kubectl get secret test-server-tls-example-com -n default
kubectl describe secret test-server-tls-example-com -n default
```

#### 인증서 내용 확인

**인증서 상세 정보 확인**

```bash
kubectl get secret test-server-tls-example-com -n default -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -text
```

**인증서 유효기간 확인**

```bash
kubectl get secret test-server-tls-example-com -n default -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -dates
```

#### Secret 구조

발급된 인증서 Secret은 다음과 같은 구조를 갖습니다.

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded certificate>
  tls.key: <base64-encoded private key>
  ca.crt: <base64-encoded CA certificate chain>
```

### 인증서 자동 갱신

cert-manager는 인증서 만료가 임박하면 자동으로 갱신을 수행합니다.

#### 자동 갱신 동작 방식

- cert-manager는 주기적으로 Certificate 리소스의 만료 시간을 확인합니다.
- `renewBefore` 필드에 설정된 시간만큼 만료일이 남았을 때 자동으로 갱신을 시작합니다.
- 갱신된 인증서는 동일한 Secret에 자동으로 업데이트됩니다.

#### 갱신 주기 설정

Certificate 리소스의 `renewBefore` 필드를 수정하여 갱신 시작 시점을 조정할 수 있습니다.

```yaml
spec:
  renewBefore: 720h  # 30일 전에 갱신 시작
```

#### 수동 갱신

필요 시 Certificate 리소스를 수동으로 갱신할 수 있습니다.

**방법 1: cmctl 명령어 사용**

```bash
# cmctl 설치(macOS)
brew install cmctl

# 인증서 갱신
cmctl renew test-server-cert-example-com -n default
```

**방법 2: kubectl annotation 사용**

```bash
# Certificate annotation을 통한 수동 갱신 트리거
kubectl annotate certificate test-server-cert-example-com -n default \
  cert-manager.io/issue-temporary-certificate="true" --overwrite
```

**방법 3: Certificate 리소스 재생성**

```bash
kubectl delete certificate test-server-cert-example-com -n default
kubectl apply -f test-server-cert-example-com.yml
```

!!! tip "알아두기"
    - cert-manager는 기본적으로 만료 30일 전부터 갱신을 시도합니다.
    - `renewBefore` 값을 너무 짧게 설정하면 인증서가 만료될 위험이 있으므로 주의해야 합니다.
    - 갱신 실패 시 cert-manager는 자동으로 재시도합니다.

#### 갱신 테스트 방법

자동 갱신이 제대로 동작하는지 테스트하려면 다음 방법을 사용할 수 있습니다.

**방법 1: 자동 갱신 시간까지 대기**

Base 인증서 발급 시 TTL을 짧게 설정하고, Certificate의 `renewBefore` 값도 짧게 설정하여 빠르게 갱신 프로세스를 확인할 수 있습니다.

예시 설정
- Base 인증서 TTL: 2일
- Certificate `renewBefore`: 24h(만료 24시간 전에 갱신 시작)

```yaml
spec:
  renewBefore: 24h  # 만료 24시간 전에 갱신 시작
```

**방법 2: 수동 갱신으로 즉시 테스트**

위의 수동 갱신 방법(cmctl 또는 kubectl annotation)을 사용하여 즉시 갱신 프로세스를 테스트할 수 있습니다.

```bash
# cmctl을 이용한 갱신
cmctl renew test-server-cert-example-com -n default
```

### 애플리케이션에서 인증서 사용

발급된 인증서는 Kubernetes Secret으로 저장되므로 다양한 방식으로 애플리케이션에 마운트할 수 있습니다.

#### Ingress에서 사용

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
    secretName: test-server-tls-example-com  # Certificate에서 생성한 Secret
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

#### Pod에서 Volume으로 마운트

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

마운트된 인증서는 다음 경로에서 사용할 수 있습니다.

- `/etc/tls/tls.crt`: 인증서
- `/etc/tls/tls.key`: 개인 키
- `/etc/tls/ca.crt`: CA 인증서 체인

### 문제 해결하기

#### 인증서 발급 실패 시

1. **Issuer 상태 확인**

```bash
kubectl describe issuer my-acme-issuer-example-com -n default
```

- `Ready` 상태가 `True`인지 확인합니다.
- ACME 계정 등록이 성공했는지 확인합니다.

2. **Certificate 상태 확인**

```bash
kubectl describe certificate test-server-cert-example-com -n default
```

- Events 섹션에서 오류 메시지를 확인합니다.

3. **CertificateRequest 확인**

```bash
kubectl get certificaterequest -n default
kubectl describe certificaterequest <request-name> -n default
```

4. **Challenge 확인**

```bash
kubectl get challenge -n default
kubectl describe challenge <challenge-name> -n default
```

- HTTP-01 Challenge의 경우 Ingress가 올바르게 설정되었는지 확인합니다.
- Challenge URL에 접근 가능한지 확인합니다.

#### 일반적인 오류와 해결 방법

| 오류 메시지 | 원인 | 해결 방법 |
|-----------|------|----------|
| `Failed to register ACME account` | EAB 정보가 잘못되었습니다. | `keyID`와 Secret 값을 확인합니다. |
| `challenge failed` | Challenge 검증에 실패했습니다. | Ingress 설정 및 네트워크 접근성을 확인합니다. |
| `domain not allowed` | Base 인증서에 없는 도메인을 요청했습니다. | Certificate의 `commonName`과 `dnsNames`를 Base 인증서와 일치시킵니다. |
| `secret not found` | EAB Secret을 찾을 수 없습니다. | Secret이 올바른 네임스페이스에 생성되었는지 확인합니다. |
| `x509: certificate signed by unknown authority` | TLS 검증 실패. | Issuer에 `skipTLSVerify: true`를 설정합니다. |

#### 로그 확인

cert-manager의 상세 로그를 확인하여 문제를 진단할 수 있습니다.

```bash
kubectl logs -n cert-manager deployment/cert-manager -f
```

!!! danger "주의"
    - ED25519 키 타입은 cert-manager의 일부 버전에서 지원하지 않을 수 있습니다. 인증서 체인에 ED25519 알고리즘을 사용하는 인증서가 포함된 경우 발급 및 갱신이 불가능할 수 있습니다.
    - 인증서 체인은 최소 3단 이상(Root → Intermediate → Leaf) 구성되어야 합니다.
    - `renewBefore` 값은 ACME 서버의 Rate Limit 정책을 고려하여 신중히 조정해야 합니다.

## ACME 프로토콜 정보

Private CA에서 제공하는 ACME Directory URL(`/directory`)을 통해 ACME 클라이언트는 필요한 모든 엔드포인트 정보를 자동으로 가져옵니다.

ACME 프로토콜의 전체 흐름은 클라이언트에 의해 자동으로 처리되므로, 사용자는 Directory URL만 제공하면 됩니다.

ACME 프로토콜에 대한 자세한 내용은 [RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555)를 참고하세요.

## 참고 자료

- [cert-manager 공식 문서](https://cert-manager.io/docs/)
- [cert-manager ACME 설정 가이드](https://cert-manager.io/docs/configuration/acme/)
- [ACME 프로토콜 명세(RFC 8555)](https://datatracker.ietf.org/doc/html/rfc8555)
- [Let's Encrypt - Challenge Types](https://letsencrypt.org/docs/challenge-types/)
- [Kubernetes Ingress TLS 설정](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)
- [ACME 인증서 갱신 가이드(Certbot, acme.sh)](acme-guide.md)
