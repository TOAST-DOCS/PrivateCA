# ACME Certificate Renewal Guide (cert-manager)
**Management > Private CA > ACME Certificate Renewal Guide (cert-manager)**

The Private CA service supports the automatic certificate management environment (ACME) protocol, enabling automated issuance and renewal of certificates. Cert-manager is a Kubernetes-native controller that automates certificate management, allowing for issuance and periodic renewal without manual intervention.

This guide explains how to use a Private CA ACME server with cert-manager to issue and automatically renew certificates within a Kubernetes environment.

!!! tip "Notice"
   - **Automatic Certificate Management Environment (ACME)**: A standard protocol for automating certificate issuance and renewal.
   - **cert-manager**: The native controller that automatically manages TLS certificates in Kubernetes.
   - **Base certificate**: The certificate in the template role that ACME auto-renewal references.
   - **certificate signing request (CSR)**: A signing request file for requesting a certificate to be issued.
   - **External account binding (EAB)**: Account binding information for authenticating to the ACME server.

!!! tip "Notice"
    To manage certificates using Certbot or acme.sh in a standard server environment, refer to the [ACME Certificate Renewal Guide (Certbot, acme.sh)](acme-guide.md).

## Prepare in advance

Before you can begin issuing certificates through ACME, you need to prepare the following:

### 1. Issue a base certificate

The base certificate acts as a "template" that the ACME server references for automatic renewal.

- Only certificates from the same domain as the domain (CN, SAN) set in the base certificate can be renewed through ACME.
- Base certificates are created in the console with the normal certificate issuance procedure.
- After you've been issued a base certificate, use the ID from that certificate in your ACME Directory URL.

### 2. Verify ACME server information

In the Private CA console, verify the following information:

- **ACME Directory URL**: `https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory`
- **ACME Token ID**: ACME token ID issued from the console (**YOUR\_ACME\_TOKEN\_ID**)
- **ACME HMAC Key**: Console-issued ACME token HMAC key (**YOUR\_ACME\_TOKEN\_HMAC\_KEY**)

## Renew Certificates with cert-manager

In a Kubernetes environment, you can use cert-manager to automatically issue and renew certificates.

### Install cert-manager

Install cert-manager on your Kubernetes cluster.

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.19.1/cert-manager.yaml 
```

**Verify Installation**

```bash
kubectl get pods -n cert-manager
```

### Install Ingress Controller

The HTTP-01 Challenge method requires an Ingress Controller.

**Install ingress-nginx**

```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

**Verify Installation**

```bash
kubectl get pods -n ingress-nginx
```

### Create an EAB Secret

Create a Kubernetes Secret containing the External Account Binding (EAB) information for ACME authentication.

```bash
kubectl create secret generic acme-eab-secret \
  --from-literal=secret='YOUR_ACME_TOKEN_HMAC_KEY' \
  --namespace default
```

!!! danger "Caution"
    -`YOUR_ACME_TOKEN_HMAC_KEY`must be replaced with the ACME HMAC key issued in Private CA Console > ACME Management.
    - The EAB Secret is sensitive information and must be managed securely.
    - The namespace where the Secret is generated must be the same as the namespace where the Issuer will be located.

### Issuer Settings

An Issuer is a cert-manager resource that defines the Certificate Authority (CA) for certificate issuance. You can choose between an Issuer, which operates at the namespace level, and a ClusterIssuer, which is available cluster-wide.

You can choose to use either the **Ingress** or **Gateway API** as the HTTP-01 Challenge validation method.

#### Method 1: Configure an Issuer with Ingress

An example of an Issuer configuration when using an Ingress Controller.

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: my-acme-issuer-example-com
  namespace: default
spec:
  acme:
    server: https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory

    # --- Start adding EAB settings --- --- External Account Binding.
    externalAccountBinding:
      # keyID: "9998617a-2799-41dd-a75b-ff093f5472c7" # Example: "kid-1", "my-account-id", etc. ID received from the server.
      keySecretRef:
        name: acme-eab-secret # Secret name created above.
        key: secret # Key name inside the Secret
    # --- End of adding EAB settings

    privateKeySecretRef:
      name: my-acme-account-key-example-com # Generated automatically by cert-manager.
    skipTLSVerify: true # required as this is a private certificate server
    solvers:
    - http01:
        ingress:
          class: nginx
```

#### Method 2: Configure an Issuer with Gateway API

An Issuer configuration example for use with the Kubernetes Gateway API.

**Prerequisites**

1. **Install Gateway API**

```bash
kubectl apply -f "https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml"
```

2. **Enable Gateway API function for cert-manager**

**If you installed with Helm**

```bash
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set "extraArgs={--enable-gateway-api}"
```

**If you installed with Manifest**

```bash
kubectl edit deployment cert-manager -n cert-manager
```

Add the following:

```yaml
spec:
  template:
    spec:
      containers:
      - name: cert-manager
        args:
        - --enable-gateway-api
```

3. **Install Traefik (Gateway Controller example)**

```bash
# Add a Helm repository
helm repo add traefik https://traefik.github.io/charts

# Install Traefik
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

# Automatically Generate a GatewayClass
gatewayClass: enabled: true name: traefik

# Disable Automatic Creation of Gateway (create them manually)
gateway: enabled: false

# Set Internal Port to 1024 or Higher
ports:
  web:
    port: 8000 # internal port
    exposedPort: 80 # external port
  websecure:
    port: 8443 # internalport
    exposedPort: 443 # external port

service:
  type: LoadBalancer

ingressRoute:
  dashboard:
    enabled: true

logs:
  general:
    level: INFO
```

4. **Create a Gateway resource**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: traefik
  namespace: default
spec:
  gatewayClassName: traefik
  listeners:
  # HTTP Listener (port 80)
  - name: http
    protocol: HTTP
    port: 8000 # Matches Traefik's web entryPoint
    allowedRoutes:
    namespaces:
      from: All
  # HTTPS Listener (port 443)
  - name: https
    protocol: HTTPS
    port: 8443 # Matches Traefik's websecure entryPoint
    allowedRoutes:
      namespaces:
        from: All
    tls:
      mode: Terminate
      certificateRefs:
      - kind: Secret
        name: test-server-tls-example-com  # TLS Secret name
        namespace: default
```

Apply Gateway.

```bash
kubectl apply -f gateway.yml
```

**Configure an Issuer with Gateway API**

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: my-acme-issuer-example-com
  namespace: default
spec:
  acme:
    server: https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory

    # --- Start adding EAB settings --- --- External Account Binding.
    externalAccountBinding:
      keyID: "9998617a-2799-41dd-a75b-ff093f5472c7"
      keySecretRef:
        name: acme-eab-secret
        key: secret
    # --- End of adding EAB settings ---.

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

!!! tip "Notice"
    Using the Gateway API, cert-manager automatically provisions HTTPRoute, Service, and Pod resources to address the challenge. Please note that the Gateway resource must be created in advance.

#### Key Field Description

| Fields | Description | Required |
|------|------|------|------|
| `spec.acme.server` | ACME Directory URL. Contains the Private CA Base certificate ID. | O |
| `spec.acme.externalAccountBinding.keyID` | ACME Token ID. Enter the value issued by the Private CA console. | O |
| `spec.acme` `.` `externalAccountBinding.keySecretRef.name` | Secret name where the EAB HMAC key is stored. | O |
| `spec.acme.externalAccountBinding.keySecretRef.key` | Key name inside the Secret. | O |
| `spec.acme.privateKeySecretRef.name` | Name of the Secret to store the ACME account private key. It is automatically generated by cert-manager. | O |
| `spec.acme.skipTLSVerify` | Whether to skip TLS certificate verification. Set `to true`in private certificate environments. | X |
| `spec.acme.solvers` | ACME Challenge verification method. Choose from `http01`, `dns01`. | O |
| `spec.acme.solvers.http01.ingress.class` | Ingress method: Ingress Controller class name (e.g. `nginx`). | X |
| `spec.acme.solvers.http01.gatewayHTTPRoute.parentRefs` | Gateway API method: Gateway resource information to reference. | X |

#### Apply Issuer and Check Status

Apply the Issuer resource.

```bash
kubectl apply -f my-acme-issuer-example-com.yml
```

Check the Issuer status.

```bash
kubectl get issuer -n default
kubectl describe issuers.cert-manager.io my-acme-issuer-example-com -n default
```

If the `Ready` status shows `True`, it's successfully enrolled.

!!! tip "Notice"
    - Ingress method: Ingress Controller must be installed.
    - Gateway API method: requires Gateway API CRD installation, enabling the Gateway API feature in cert-manager, installing a Gateway Controller (e.g., Traefik), and creating a Gateway resource.
    - `dns01` solver requires DNS provider setup.
    - The `skipTLSVerify: true` option is required if the Private CA server uses private certificates.

#### Configure hostAliases for the HTTP-01 Challenge (Preliminary Work)

Before creating a Certificate, you must ensure that your environment is properly configured for a successful HTTP-01 challenge.

The HTTP-01 challenge requires cert-manager to send an HTTP request to your domain to verify ownership. In test environments where the domain is not registered in public DNS, you must configure hostAliases in the cert-manager Deployment to map the domain to the IP address of your Kubernetes cluster.

```bash
kubectl edit deployment cert-manager -n cert-manager
```

Add the following content under `spec.template.spec`:

```yaml
spec:
  template:
    spec:
      hostAliases:
      - ip: "114.110.145.74" # External IP of the Kubernetes cluster (e.g., NKS Floating IP, LoadBalancer IP)
      hostnames:
      - "example.com" # Any domain you set in the certificate
      - "sub.example.com"
```

This configuration ensures that requests from the cert-manager Pod to the domain are routed to the specified IP address.

!!! danger "Caution"
    - The IP address must be set to the external IP of the Ingress Controller of the Kubernetes cluster or the LoadBalancer of the Gateway.
    - Make sure to add **all the domains**`(commonName` and `dnsNames`) you set in the certificate to `the hostnames`.
    - In production environments, it is recommended to set up real DNS records.

!!! tip "Notice" When using the Gateway API, cert-manager automatically creates the following resources to handle the challenge process.

    1. **HTTPRoute**: For each domain, create an HTTPRoute to handle the Challenge path (`/.well-known/acme/v2.0-challenge/`).
    2. **Service**: Create a Service to handle the Challenge.
    3. **Pod**: Create a Pod to respond to the challenge.

    These resources are automatically deleted after the Challenge completes.

### Create a Certificate Resource

The Certificate resource defines the properties of the certificate to be issued.

#### Example of Certificate Configuration

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: test-server-cert-example-com
  namespace: default
spec:
  secretName: test-server-tls-example-com # Secret where the certificate will be stored
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

#### Key Field Description

| Fields | Description | Required |
|------|------|------|
| `spec.secretName` | Secret name to store the issued certificate and private key. | O |
| `spec.renewBefore` | Time to start renewal before the certificate expires. Example: `360h`(15 days). | X |
| `spec.commonName` | Common Name (CN) of the certificate. Must match the CN of the base certificate. | O |
| `spec.dnsNames` | Subject Alternative Names (SAN) of the certificate. Must match the SAN of the base certificate. | X |
| `spec.issuerRef.name` | Issuer or ClusterIssuer name to use. | O |
| `spec.issuerRef.kind` | Issuer type. `Issuer` or `ClusterIssuer`. | X |

#### Apply the Certificate and Check its Status

Apply a Certificate resource.

```bash
kubectl apply -f test-server-cert-example-com.yml
```

Check the certificate status.

```bash
kubectl get certificate -n default
kubectl describe certificate test-server-cert-example-com -n default
```

If the certificate issuance is successful, the `Ready` status is displayed as `True`.

**View Detailed Certificate Issuance Progress**

```bash
# Verify CertificateRequest (Create Order)
kubectl get certificaterequest -n default

# Confirm an Order (Fulfillment)
kubectl get order -n default

# Check Challenge (Challenge Verification)
kubectl get challenge -n default
```

!!! danger "Caution"
    - The domains specified `in` `commonName and` `dnsNames`must exactly match the CN and SAN set in the Base certificate.
    - If you add a domain that does not exist in the base certificate, the certificate will fail to issue.
    - Be sure to verify that you specified the correct domain by checking the CN and SAN information in the Base certificate in the console before issuing the certificate.

### Verify Issued Certificates

Once the certificate is successfully issued, it will be stored in the designated Secret.

#### Confirm Secret

```bash
kubectl get secret test-server-tls-example-com -n default
kubectl describe secret test-server-tls-example-com -n default
```

#### Verify Certificate Contents

**View Certificate Details**

```bash
kubectl get secret test-server-tls-example-com -n default -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -text
```

**Verify Certificate Validity**

```bash
kubectl get secret test-server-tls-example-com -n default -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -noout -dates
```

#### Secret Structure

The issued certificate Secret has the following structure:

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded certificate>
  tls.key: <base64-encoded private key>
  ca.crt: <base64-encoded CA certificate chain>
```

### Certificate Auto-Renewal

cert-manager automatically performs renewals when a certificate is nearing expiration.

#### How Auto-Renewal Works

- The cert-manager periodically checks the expiration time of certificate resources.
- Automatically starts renewal when the time set in the `renewBefore` field is up to the expiration date.
- Renewed certificates are automatically updated to the same Secret.

#### Set Renewal Cycle

You can adjust when renewals start by modifying the `renewBefore` field in the certificate resource.

```yaml
spec:
  renewBefore: 720h # Start renewal 30 days ago
```

#### Manual Renewal

You can manually renew certificate resources as needed.

**Method 1: Use the cmctl command**

```bash
# Install cmctl (macOS)
brew install cmctl

# Renew Certificate
cmctl renew test-server-cert-example-com -n default
```

**Method 2: Use kubectl annotations**

```bash
# Trigger Manual Renewals with Certificate Annotations
kubectl annotate certificate test-server-cert-example-com -n default \
    cert-manager.io/issue-temporary-certificate="true" --overwrite
```

**Method 3: Regenerate a certificate resource**

```bash
kubectl delete certificate test-server-cert-example-com -n default
kubectl apply -f test-server-cert-example-com.yml
```

!!! tip "Notice"
    - cert-manager will attempt to renew 30 days before expiration by default.
    - Be careful not to set the `renewBefore` value too short, as you risk letting the certificate expire.
    - If the renewal fails, cert-manager will automatically retry.

#### How to Test for Renewals

To test that auto-renewal is working properly, you can use the following methods.

**Method 1: Wait until the auto-renewal time**

You can quickly verify the renewal process by setting a short TTL for the Base certificate and a smaller `renewBefore` value in the Certificate resource.

Example settings - Base certificate TTL: 2 days - Certificate `renewBefore`: 24h (Start renewal 24 hours before expiration)

```yaml
spec:
  renewBefore: 24h # Start renewal 24 hours before expiration
  ```

**Method 2: Test immediately with manual renewal**

You can use the manual update method above (cmctl or kubectl annotation) to test the update process out of the box.

```bash
# Renewal with cmctl
cmctl renew test-server-cert-example-com -n default
```

### Using certificates in applications

The issued certificate is stored as a Kubernetes Secret, allowing you to mount it in your application in various ways.

#### Used by Ingress

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
    secretName: test-server-tls-example-com  # Secret rules created by the certificate
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

#### Mount from Pod to Volume

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

Mounted certificates are available at the following paths:

- `/etc/tls/tls.crt`: Certificates
- `/etc/tls/tls.key`: Private key
- `/etc/tls/ca.crt`: CA Certificate Chain

### Troubleshooting

#### If certificate issuance fails

1. **Check Issuer status**

```bash
kubectl describe issuer my-acme-issuer-example-com -n default
```

- Verify that the `Ready` status `is True`.
- Verify that your ACME account registration was successful.

2. **Check Certificate Satus**

```bash
kubectl describe certificate test-server-cert-example-com -n default
```

- In the Events section, check for error messages.

3. **Verify ertificateRequest**

```bash
kubectl get certificaterequest -n default
kubectl describe certificaterequest <request-name> -n default
```

4. **Check Challenge**

```bash
kubectl get challenge -n default
kubectl describe challenge <challenge-name> -n default
```

- For HTTP-01 Challenge, make sure Ingress is configured correctly.
- Verify that the Challenge URL is accessible.

#### Common Errors and Solutions

| Error Message | Cause | Solution |
|-----------|------|----------|
| `Failed to register ACME account` | EAB information is incorrect. | Check the `keyID and`Secret values. |
| `challenge failed` | Challenge verification failed. | Check the Ingress settings and network accessibility. |
| `domain not allowed` | Requested a domain that is not in the base certificate. | Match the `commonName and` `dnsNames`of the certificate to the base certificate. |
| `secret not found` | EAB Secret not found. | Verify that the Secret was created in the correct namespace `. |
| x509: certificate signed by unknown authority` | TLS verification failed. | Set `skipTLSVerify: true`for the Issuer. |

#### Check Logs

You can diagnose the problem by checking the detailed logs in cert-manager.

```bash
kubectl logs -n cert-manager deployment/cert-manager -f
```

!!! danger "Caution"
    - The ED25519 key type may not be supported by all versions of cert-manager. If the certificate chain contains certificates that use the ED25519 algorithm, issuance and renewal might not be possible.
    - The certificate chain must consist of at least three levels (Root → Intermediate → Leaf).
    - The `renewBefore` value must be carefully adjusted to account for the ACME server's Rate Limit policy.

## About ACME protocol

Through the ACME Directory URL `(/directory`) provided by the private CA, the ACME client automatically gets all the endpoint information it needs.

The ACME protocol workflow is fully automated by the client, requiring only the Directory URL from the user.

For more information about the ACME protocol, see [RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555).

## References

- [cert-manager official documentation](https://cert-manager.io/docs/)
- [cert-manager ACME Configuration Guide](https://cert-manager.io/docs/configuration/acme/)
- [ACME Protocol Specification (RFC 8555)](https://datatracker.ietf.org/doc/html/rfc8555)
- [Let's Encrypt - Challenge Types](https://letsencrypt.org/docs/challenge-types/)
- [Configure Kubernetes Ingress TLS](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)
- [ACME Certificate Renewal Guide (Certbot, acme.sh)](acme-guide.md)
