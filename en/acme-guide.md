# Guide to ACME Certificate Renewal
**Management > Private CA > Guide to ACME Certificate Renewal**

Private CA service supports the automatic certificate management environment (ACME) protocol to enable the automatic issuance and renewal of certificates. ACME clients (e.g: Certbot) allow you to issue and periodically renew certificates without manual intervention.

This guide walks you through how to use the Private CA ACME server to issue certificates with Certbot.

!!! tip "Notice"
    - **Automatic certificate management environment (ACME)**: A standard protocol for automating certificate issuance and renewal.
    - **Certbot**: An ACME client tool developed by Let's Encrypt that automatically issues and renews certificates.
    - **Base certificate**: The certificate in the template role that ACME auto-renews references.
    - **Certificate signing request (CSR)**: a file used to request certificate issuance.
    - **External account binding (EAB**): Account binding information to authenticate to the ACME server.

## Prepare in advance

Before you can begin issuing certificates through ACME, you need to prepare the following:

### 1. Issue a Base Certificate

The base certificate acts as a "template" that the ACME server references for automatic renewal.

- Only certificates from the same domain as the domain (CN, SAN) set in the base certificate can be renewed through ACME.
- Base certificates are created in the console with the normal certificate issuance procedure.
- After you've been issued a base certificate, use the ID from that certificate in your ACME Directory URL.

### 2. Install Certbot

Certbot is the most widely used ACME client. Refer to the [official Certbot documentation](https://certbot.eff.org/) to install it for your operating system.

**Installation example (Ubuntu/Debian):**

```bash
sudo apt update
sudo apt install certbot
```

**Installation example (CentOS/RHEL):**

```bash
sudo yum install certbot
```

### 3. Verify ACME server information

In the Private CA console, verify the following information:

- **ACME Directory URL**: `https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory`
- **ACME token ID**: ACME token ID issued from the console**(YOUR_ACME_TOKEN_ID**)
- **ACME HMAC key**: Console-issued ACME token**HMAC** key **(YOUR_ACME_TOKEN_HMAC_KEY**)

## Renew a Certificate

The process for using Certbot to issue a certificate is as follows:

### Configure commands

Here's an example of a basic certificate issuance command:

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

### Key Option Description

| Options | Description | Required | Default |
|------|------|------|--------|
| `--manual` | Perform Challenge verification manually. Use manual mode instead of the Apache or Nginx automatic settings: | O | `- |
| `--manual-auth-hook` | Specifies a script to run before Challenge verification. Even an empty script is required to prevent errors at renewal time. | O | - |
| `--deploy-hook` | Specifies a script to run after successful certificate issuance. Use for post-processing, such as certificate movement, deployment, etc. | X | - |
| `--preferred-challenges` | Specifies the challenge method`(http`, `dns`, `tls-alpn`). Typically, this uses `http`. | O | - |
| `--server` | Specifies the Directory URL of the ACME server. | O | - |
| `-d` | Specifies the domain to include in the certificate. The CN and SAN of the base certificate must be verified in the console and entered correctly: | O | - |
| `--eab-kid` | The key ID for External Account Binding. Enter the ACME token ID issued by the private CA: | O | - |
| `--eab-hmac-key` | The EAB HMAC key (Base64 encoded). Enter the ACME token HMAC key issued by the private CA | O | - |
| `--key-type` | Specifies the private key type`(rsa`, `ecdsa`) | X | rsa |
| `--rsa-key-size` | Specifies the RSA key length. Used only if `--key-type is rsa`, values `2048`, `3072`, and `4096` are acceptable. | X | 2048 |
| `--elliptic-curve` | Specifies the ECDSA key curve. Use only if `--key-type ecdsa`, values `secp256r1`, `secp384r1, secp521r1` are acceptable. | X | secp256r1 |
| `--agree-tos` | Automatically agree to the Terms of Service. Prevents interactive input. | X | `- |
| `--register-unsafely-without-email` | Registers an account without email. Prevents interactive input. | X | - |

!!! tip "Notice"
    - The `--manual` mode is used for manual challenge validation.
    - `The manual-auth-hook` is required for reliability when renewing certificates.
    - The `deploy-hook` allows you to automatically perform deployment and post-processing tasks after certificate issuance.
    - The `--server`, `--eab-kid`, `--eab-hmac-key` are required for Private CA ACME server integration.

!!! danger "Caution"
    When specifying a domain, you must enter the common name (CN) and domain subject alternative name (SAN) set in the base certificate exactly. Before issuing the certificate, be sure to verify that you specified the correct domain in the `-d` option by checking the CN and SAN information for the base certificate in the console.

### Hook script example

#### pre.sh (pre-authentication execution script)

The `manual-auth-hook` must exist as a file, even if its contents are empty. This is to prevent errors when renewing the certificate.

```bash
#!/bin/bash
# You can add any necessary authentication preprocessing tasks here.
```

#### post.sh (script to execute after certificate issuance)

The `deploy-hook` is only executed if the certificate is successfully issued.

```bash
#!/bin/bash
# Processing tasks after certificate issuance is complete
echo "[post.sh] Certificate issued successfully!"

# Copy the certificate to the target location
cp /etc/letsencrypt/live/example.com/fullchain.pem ~/Downloads/
cp /etc/letsencrypt/live/example.com/privkey.pem ~/Downloads/

# Additional tasks, such as restarting the web server
# systemctl reload nginx
```

## Verify Issued Certificates

Certificates are stored in the following paths by default:

```
/etc/letsencrypt/live/<domain name (CN)>/
├── cert.pem # server certificate
├── chain.pem # intermediate certificate chain
├── fullchain.pem # cert.pem + chain.pem
└── privkey.pem # private key
```

### Verify Certificate Contents

```bash
# Verify Certificate Information
openssl x509 -in /etc/letsencrypt/live/<domain name (CN)>/cert.pem -text -noout

# Verify Certificate Validity
openssl x509 -in /etc/letsencrypt/live/<domain name (CN)>/cert.pem -noout -dates
```

## Set up Certificate Auto-renewal

Certbot can automatically renew certificates that are nearing expiration.

### Renewal Prerequisites

- Must have an original issuance history.
- The file `/etc/letsencrypt/renewal/<domain>.conf` must exist.
- The certificate files should be located in the `/etc/letsencrypt/live/<domain>/` directory.

### Set up Auto-renewal

When you install Certbot, it automatically registers a cron or systemd timer to periodically check for certificate expiration.

**Default renewal cycle**: Attempt to auto-renew 30 days before expiration

### Manual renewal

If necessary, you can perform a manual renewal with the following commands:

```bash
# All certificate renewal attempts
sudo certbot renew

# Force renewal
sudo certbot renew --force-renewal
```

### Register a Cron job

If auto-renewal is not registered, you can manually add a cron job.

```bash
# Edit crontab
sudo crontab -e

# Check renewals at 2 AM every day
0 2 * * * certbot renew --no-random-sleep-on-renew
```

### Change a renewal cycle

You can adjust the renewal interval in the `/etc/letsencrypt/renewal/<domain>.conf` file.

```ini
[renewalparams] renew_before_expiry = 30 days
```

You can change the `renew_before_expiry` value to set how many days before the certificate expires to attempt to renew.

!!! danger "Caution"
    - The ED25519 key type is not currently supported by Certbot. If a certificate chain contains certificates that use the ED25519 algorithm, they can't be issued or renewed.
    - The certificate chain must consist of at least three levels (Root → Intermediate → Leaf).
    - The `renew_before_expiry` value must be carefully adjusted to account for the ACME server's Rate Limit policy.
    - The auto-registered cron jobs that are included with the Certbot installation may contain default options, so it is recommended that you modify the `/etc/cron.d/certbot` file as needed.
    - The default cron job includes a random delay `(perl -e 'sleep int(rand(43200))')`. This is to prevent overloading the ACME server, and if you need it to execute immediately, you should remove the syntax or use the `--no-random-sleep-on-renew` option.

## Troubleshooting

### If certificate issuance fails

1. **Verify** the **ACME Directory URL**: verify that the URL in the `--server` option is correct.
2. **Verify EAB credentials**: verify that the `--eab-kid and` `--eab-hmac-key` values are correct.
3. **Domain verification failed**: Verify that the Challenge method is suitable for your environment. For HTTP Challenge, port 80 must be open.
4. **Hook script permissions**: ensure that the `pre.sh and` `post.sh` files have execute permissions.

### If certificate renewal fails

1. **Verify Renewal settings**: verify that the `/etc/letsencrypt/renewal/<domain>.conf` file exists and is correct.
2. **Check hook script existence**: verify that the script specified `with manual-auth-hook` still exists.

## About ACME protocol

Through the ACME Directory URL `(/directory`) provided by the private CA, the ACME client automatically gets all the endpoint information it needs.

The ACME protocol workflow is fully automated by the client, requiring only the Directory URL from the user.

For more information about the ACME protocol, see [RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555).

## References

- [Certbot official documentation](https://certbot.eff.org/)
- [ACME Protocol Specification (RFC 8555)](https://datatracker.ietf.org/doc/html/rfc8555)
- [Let's Encrypt - Challenge Types](https://letsencrypt.org/docs/challenge-types/)
