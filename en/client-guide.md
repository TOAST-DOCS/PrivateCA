# ACME Certificate Renewal Guide (Certbot, acme.sh)
**Management > Private CA > ACME Certificate Renewal Guide (Certbot, acme.sh)**

The Private CA service supports the automatic certificate management environment (ACME) protocol, enabling automated issuance and renewal of certificates. ACME client (e.g., Certbot, acme.sh) allows you to issue and periodically renew certificates without manual intervention.

This guide explains how to use a Private CA ACME server with Certbot or acme.sh to issue and automatically renew certificates.

!!! tip "Notice"
    - **Automatic Certificate Management Environment (ACME)**: A standard protocol for automating certificate issuance and renewal.
    - **Certbot**: An ACME client tool developed by Let's Encrypt that automatically issues and renews certificates.
    - **acme.sh**: A lightweight ACME client written in pure Unix Shell that enables automated certificate issuance and renewal.
    - **Base Certificate**: The certificate in the template role that ACME auto-renews references.
    - **Certificate Signing Request (CSR)**: It is used to request certificate issuance.
    - **EAB(external account binding)**: Account binding information to authenticate to the ACME server.

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

## Renew a Certificate

You can use either Certbot or acme.sh as your ACME client. Please choose the tool that best fits your environment to proceed.

- **Certbot**: An ACME client tool developed by Let's Encrypt, based on Python.
- **acme.sh**: A lightweight ACME client written in pure Shell script with minimal dependencies and an easy installation process.

!!! tip "Notice" To automate certificate management in Kubernetes, please refer to the [ACME Certificate Renewal Guide](cert-manager-guide.md) (cert-manager).

### Renew your certificate with Certbot

#### Install Certbot

Certbot is the most widely used ACME client. Refer to the [official Certbot documentation](https://certbot.eff.org/) to install it for your operating system.

**Installation example (Ubuntu/Debian)**

```bash
sudo apt update
sudo apt install certbot
```

**Installation example (CentOS/RHEL)**

```bash
sudo yum install certbot
```

#### Configure commands

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

#### Key Option Description

| Option | Description | Required | Default |
|------|------|------|--------|
| `--manual` | Manually perform challenge verification. Use manual mode instead of Apache or Nginx automatic configuration. | O | - |
| `--manual-auth-hook` | Specifies a script to run before challenge verification. Even an empty script is required to prevent errors during renewal. | O | - |
| `--deploy-hook` | Specifies a script to run after successful certificate issuance. Used for post-processing, such as certificate transfer and deployment. | X | - |
| --preferred-challenges | Specifies the challenge method ( `http`, `dns`, `tls-alpn` ). Typically, `http` is used. | O | - |
| `--server` | Specifies the Directory URL of the ACME server. | O | - |
| `-d` | Specifies the domains to include in the certificate. The CN and SAN of the base certificate must be entered correctly, as verified in the console. | O | - |
| `--eab-kid` | Key ID for External Account Binding. Enter the ACME token ID issued by the Private CA. | O | - |
| `--eab-hmac-key` | EAB HMAC key (Base64 encoded). Enter the ACME token HMAC key issued by the Private CA. | O | - |
| `--key-type` | Specifies the private key type ( `rsa` , `ecdsa` ). | X | rsa |
| `--rsa-key-size` | Specifies the RSA key length. Only used when --key-type is rsa . Valid values ​​are `2048`, `3072`, and `4096`. | X | 2048 |
| `--elliptic-curve` | Specifies the ECDSA key curve. Only used when it is `--key-type ecdsa` . Valid values ​​are `secp256r1`, `secp384r1`, and `secp521r1`. | X | secp256r1 |
| `--agree-tos` | Automatically agree to the Terms of Service. Prevents interactive input. | X | - |
| `--register-unsafely-without-email` | Register an account without an email. Prevents interactive input. | X | - |

!!! tip "Notice"
    - The `--manual` mode is used for manual challenge validation.
    - `The manual-auth-hook` is required for reliability when renewing certificates.
    - The `deploy-hook` allows you to automatically perform deployment and post-processing tasks after certificate issuance.
    - The `--server`, `--eab-kid`, `--eab-hmac-key` are required for Private CA ACME server integration.

!!! danger "Caution"
    When specifying domains, you must ensure they match the Common Name (CN) and Subject Alternative Name (SAN) of the Base certificate. Before issuance, please verify the certificate details in the console to confirm that the correct domains are assigned to the `-d` option.

#### Hook script example

##### pre.sh (pre-authentication execution script)

`The manual-auth-hook` must exist as a file, even if its contents are empty. This is to prevent errors during certificate renewal.

```bash
#!/bin/bash
# You can add any necessary authentication preprocessing tasks here.
```

##### post.sh (script to execute after certificate issuance)

The `deploy-hook` is only executed if the certificate is successfully issued.

```bash
#!/bin/bash
# Processing tasks after certificate issuance is complete
echo "\[post.sh] Certificate issued successfully!"

# Copy the certificate to the target location
cp /etc/letsencrypt/live/example.com/fullchain.pem ~/Downloads/
cp /etc/letsencrypt/live/example.com/privkey.pem ~/Downloads/

# Additional tasks, such as restarting the web server
# systemctl reload nginx
```

#### Verify Issued Certificates

Certificates are stored in the following paths by default:

```
/etc/letsencrypt/live/<domain name (CN)>/
├── cert.pem # server certificate
├── chain.pem # intermediate certificate chain
├── fullchain.pem # cert.pem + chain.pem
└── privkey.pem # private key
```

##### Verify Certificate Contents

```bash
# Verify Certificate Information
openssl x509 -in /etc/letsencrypt/live/<domain name (CN)>/cert.pem -text -noout

# Verify Certificate Validity
openssl x509 -in /etc/letsencrypt/live/<domain name (CN)>/cert.pem -noout -dates
```

#### Set up Certificate Auto-renewal

Certbot can automatically renew certificates that are nearing expiration.

##### Renewal Prerequisites

- Must have an original issuance history.
- The file `/etc/letsencrypt/renewal/<domain>.conf` must exist.
- The certificate files should be located in the `/etc/letsencrypt/live/<domain>/` directory.

##### Set up Auto-renewal

When you install Certbot, it automatically registers a cron or systemd timer to periodically check for certificate expiration.

**Default renewal cycle**: Attempt to auto-renew 30 days before expiration

##### Manual renewal

If necessary, you can perform a manual renewal with the following commands:

```bash
# All certificate renewal attempts
sudo certbot renew

# Force renewal
sudo certbot renew --force-renewal
```

##### Register a Cron job

If auto-renewal is not registered, you can manually add a cron job.

```bash
# Edit crontab
sudo crontab -e

# Check renewals at 2 AM every day
0 2 * * * certbot renew --no-random-sleep-on-renew
```

##### Change a renewal cycle

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

#### Troubleshooting

##### If certificate issuance fails

1. **Verify the ACME Directory URL**: verify that the URL in the `--server` option is correct.
2. **Verify EAB credentials**: verify that the `--eab-kid and` `--eab-hmac-key` values are correct.
3. **Domain verification failed**: Verify that the Challenge method is suitable for your environment. For HTTP Challenge, port 80 must be open.
4. **Hook script permissions**: ensure that the `pre.sh and` `post.sh` files have execute permissions.

##### If certificate renewal fails

1. **Verify Renewal settings**: verify that the `/etc/letsencrypt/renewal/<domain>.conf` file exists and is correct.
2. **Check hook script existence**: verify that the script specified `with manual-auth-hook` still exists.

### Certificate Renewal with acme.sh

acme.sh is a lightweight ACME client written in pure Shell script, with minimal dependencies and an easy installation process.

#### Install acme.sh

acme.sh can be easily installed via an install script.

**Installation**

```bash
curl https://get.acme.sh | sh
```

After the installation is complete, load the environment variables.

```bash
. ~/.acme.sh/acme.sh.env acme.sh --version
```

!!! tip "Notice" - When reconnecting to a container or a new shell session, you must reload environment variables with the command `. ~/.acme.sh/acme.sh.env`. - The email address is optional during installation.

**Installation Example (Manual Download)**

```bash
git clone https://github.com/acmesh-official/acme.sh.git
cd acme.sh
./acme.sh --install
```

#### Sign up for an ACME Account

Before you can issue a certificate, you must first register an account with the ACME server.

```bash
acme.sh --register-account \
  --server https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory \
  --eab-kid "YOUR_ACME_TOKEN_ID" \
  --eab-hmac-key "YOUR_ACME_TOKEN_HMAC_KEY"
```

!!! DANGER "Caution" - Account registration **only** needs to be done **once for the first time**. - After registration, you can omit the `--eab-kid and` `--eab-hmac-key` options when issuing a certificate.

#### Issue Certificate

You can issue certificates after you sign up for an account.

**Example of Basic Issuance Commands**

```bash
acme.sh --issue \
  -d example.com -d www.example.com \
  --server https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory \
  --keylength 2048 \
  --standalone
```

#### Key Option Description

| Options | Description | Required | Default |
|------|------|------|------|--------|
| `--register-account` | Registers an account with the ACME server. Runs one time only. | O (first time) | - |
| `--issue` | ` Issues` a certificate. | O | - |
| `--server` | Specifies the Directory URL of the ACME server. | O | - |
| `--eab-kid` | Key ID for External Account Binding. Enter the ACME token ID issued by the Private CA. Required only for account registration. | O (at registration) | - |
| `--eab-hmac-key` | EAB HMAC key (Base64 encoded). Enter the ACME token HMAC key issued by the Private CA. Required only when you enroll for an account. | O (at enrollment) | - | |
| `-d` | Specifies the domains to include in the certificate. The CN and SAN of the base certificate must be verified in the console to ensure they are entered correctly. Multiple can be specified. | O | - |
| `--standalone` | Process HTTP-01 challenges in standalone mode. acme.sh runs a temporary web server. | O | `- |
| --httpport` | Specifies the port number to use for HTTP-01 challenges. | X | ` 80 |
| --keylength` | Specifies the private key length. You can use values of `2048`, `3072`, `4096` for RSA and `ec-256`, `ec-384`, `ec-521` for ECDSA. | X | 2048 |
| `--debug` | Runs in debug mode to output a detailed log. | X | - |

!!! tip "Notice"
    - `--standalone` mode causes acme.sh to run a temporary web server to handle the Challenge. Port 80 must be available.
    - After account registration, you can issue certificates without the `--eab-kid and` `--eab-hmac-key` options.

!!! danger "Caution"
    When specifying domains, you must ensure they match the Common Name (CN) and Subject Alternative Name (SAN) of the Base certificate. Before issuance, please verify the certificate details in the console to confirm that the correct domains are assigned to the `-d` option..

#### Issue Certificates in Standalone Mode

acme.sh runs a temporary web server to handle HTTP-01 challenges.

```bash
acme.sh --issue \
  -d example.com \
  --server https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory \
  --keylength 2048 \
  --standalone
```

!!! danger "Caution"
\- Port 80 must be open and available.
\- If an existing web server is using port 80, it must be temporarily stopped.
\- To use a different port, add the `--httpport` option.

#### Verify Issued Certificates

Certificates are stored in the following paths by default:

```
~/.acme.sh/<domain name (CN)>/
├── <domain name>.cer # Server certificate (leaf certificate)
├── <domain name>.key # Private key
├── ca.cer # CA chain (Intermediate + Root)
└── fullchain.cer # Full chain (leaf certificate + CA chain) ```

!!! tip "Notice"
    - `<domain name>.cer`: contains only leaf certificates
    - `fullchain.cer`: full chain of leaf certificates + intermediate certificates + root certificate
    - `ca.cer`: CA chain (intermediate certificates + root certificate)

##### Verify Certificate Contents

```bash
# Verify Certificate Information
openssl x509 -in ~/.acme.sh/example.com/example.com.cer -text -noout

# Verify Certificate Validity
openssl x509 -in ~/.acme.sh/example.com/example.com.cer -noout -dates
```

#### Install Certificate (Deployment)

acme.sh can use the `--install-cert` command to copy the certificate to the desired location and restart the web server.

**Nginx Example**

```bash
acme.sh --install-cert -d example.com \
  --key-file       /etc/nginx/ssl/privkey.pem \
  --cert-file      /etc/nginx/ssl/cert.pem \
  --fullchain-file /etc/nginx/ssl/fullchain.pem \
  --ca-file        /etc/nginx/ssl/ca.pem \
  --reloadcmd      "nginx -s reload"
```

**Apache Example**

```bash
acme.sh --install-cert -d example.com \
  --key-file       /etc/apache2/ssl/privkey.pem \
  --cert-file      /etc/apache2/ssl/cert.pem \
  --fullchain-file /etc/apache2/ssl/fullchain.pem \
  --ca-file        /etc/apache2/ssl/ca.pem \
  --reloadcmd      "systemctl reload apache2"
```

!!! tip "Notice"
    - `--key-file`: private key file storage path
    - `--cert-file`: leaf certificate storage path
    - `--fullchain-file: full` chain (leaf + CA chain) storage path
    - `--ca-file`: CA chain storage path
    - `--reloadcmd`: command to run automatically after certificate installation

#### Set up Certificate Auto-renewal

acme.sh automatically registers a cron job upon installation to check and renew certificate expirations.

##### Renewal Prerequisites

- Must have an original issuance history.
- The certificate files should be located in the `~/.acme.sh/<domain>/` directory.

##### Set up Auto-renewal

The cron job is automatically registered when you install acme.sh.

**Default renewal cycle**: Attempt to auto-renew 60 days before expiration

**Check a Cron job**

```bash
crontab -l | grep acme.sh
```

They are typically registered in the following form:

```
0 0 * * * "/home/user/.acme.sh"/acme.sh --cron --home "/home/user/.acme.sh" > /dev/null
```

##### Manual renewal

If necessary, you can perform a manual renewal with the following commands:

```bash
# All certificate renewal attempts
acme.sh --cron

# Force Renewal of Specific Certificates
acme.sh --renew -d example.com --force
```

##### Change a renewal cycle

By default, acme.sh attempts to renew 60 days before expiration. To change this value, modify the `~/.acme.sh/account.conf` file.

```bash
# Edit account.conf
vim ~/.acme.sh/account.conf
```

Add or modify the following lines:

```
Le_RenewalDays=30
```

This setting will attempt to renew 30 days before expiration.

!!! danger "Caution"
    - The ED25519 key type may not be currently supported by acme.sh. If the certificate chain contains certificates that use the ED25519 algorithm, they cannot be issued or renewed.
    - Certificate chains must consist of at least three levels (Root → Intermediate → Leaf).
    - The `Le_RenewalDays` value must be carefully adjusted to account for the ACME server's Rate Limit policy.
    - If you're using Standalone mode, you might need a script to suspend your web server because port 80 must be available for renewal.

#### Troubleshooting

##### If certificate issuance fails

1. **Verify the ACME Directory URL**: verify that the URL in the `--server` option is correct.
2. **Verify EAB Credentials**: Verify that the account is registered; if not, register the account first with the `--register-account` command.
3. **Domain Verification Failed**: Verify that port 80 is open and available. If an existing web server is using port 80, you'll need to temporarily stop it.
4. **Debug Mode**: Add the `--debug` option to view detailed logs.

```bash
acme.sh --issue \
  -d example.com \
  --server https://kr1-pca.api.nhncloudservice.com/acme/v2.0/cert/{certId}/directory \
  --keylength 2048 \
  --standalone \
  --debug
```

##### If certificate renewal fails

1. **Verify Certificate Information**: Verify that the settings used during the original issuance are still in place.
2. **Verify a Cron Job**: Verify that cron jobs are registered properly.
3. **Check Logs**: Check the `~/.acme.sh/<domain>/<domain>.log` file for error messages.
4. **Test Manual Renewal**: Diagnose the issue by attempting a manual renewal with the command `acme.sh --renew -d example.com --force --debug`.

## About ACME protocol

Through the ACME Directory URL `(/directory`) provided by the private CA, the ACME client automatically gets all the endpoint information it needs.

The ACME protocol workflow is fully automated by the client, requiring only the Directory URL from the user.

For more information about the ACME protocol, see [RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555).

## References

- [Certbot official documentation](https://certbot.eff.org/)
- [acme.sh official documentation](https://github.com/acmesh-official/acme.sh)
- [acme.sh DNS API list](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)
- [ACME Protocol Specification (RFC 8555)](https://datatracker.ietf.org/doc/html/rfc8555)
- [Let's Encrypt - Challenge Types](https://letsencrypt.org/docs/challenge-types/)
- [ACME Certificate Renewal Guide (cert-manager)](cert-manager-guide.md)
