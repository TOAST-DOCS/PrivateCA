# Overview

**Management > Private CA > Overview**

Private CA is a service that allows you to issue and manage your own certificates for use inside your organization. This service securely issues certificates without going through an authorized certificate authority and apply them to internal systems, API servers, IoT devices, and more.

## Main features

### Run your own certificate authority (CA)

* Allow you to create a Root CA and an Intermediate CA to organize your certificate hierarchy.
* Allow you to manage and operate certificate authorities to align with your organization's security policies.
* Allow you to issue certificates independently without relying on an external certificate authority.

### Automatically issue and renew certificates

* Certificate templates can be used to quickly and consistently issue certificates with the same configuration.
* Support for the automatic certificate management environment (ACME) protocol allows you to automate certificate issuance and renewal.
* Compatible with standard ACME clients like Certbot.

### Manage certificate retirement

* A certificate revocation list (CRL) periodically provides a list of revoked certificates.
* The online certificate status protocol (OCSP) allows you to quickly check the revocation status of individual certificates to the status at the time of your request.
* The service allows you to track and audit certificate revocation history.

### API support

* Allow you to manage certificates programmatically through a RESTful API.
* Provide APIs to download certificates, look up CRLs, respond to OCSPs, and more.
* Easy to integrate with automation systems.

## Configure a service

Private CA service consists of the following components:

![Private CA service structure](../pca_images/NHN Cloud_PrivateCA_overview_ko_900.png)

### Repository

* The basic unit for managing Private CA.
* Every resource belongs to a specific repository: issuers, certificate templates, certificates, ACME tokens, and so on.
* Manage CRL and OCSP settings on a per-repository basis.

### Issuer

* A certificate authority that signs and issues certificates.
* **Root CA**: a top-level certificate authority, which is a self-signed certificate. It's the starting point of all trust.
* **Intermediate CA**: an intermediate certificate authority signed by the Root CA. Used to issue the actual server certificate.

### Certificate template

* A collection of settings for issuing certificates quickly and consistently.
* Allow you to easily issue multiple certificates with the same setup, which consists of two settings:

  * **Set limits**: define restrictions when issuing certificates, such as validity periods, SAN options, and more.
  * **Common applied settings**: define settings that you want to apply to certificates in common, such as key algorithm, key usage, extended key usage, and subject information.

### Certificate

* A real, usable certificate signed by the issuer.
* It can be used for server authentication, client authentication, code signing, and more.
* Allow you to download it in PEM format and apply it to your system.

### ACME token

* Authentication information used for automatic certificate issuance over the ACME protocol.
* Allow you to integrate with ACME clients like Certbot to automatically issue and renew certificates.
* Authenticate to the ACME server using the token ID and HMAC key.

## Certificate issuance workflow

The basic flow for issuing certificates from Private CA is as follows:

1. **Create a repository**: Create a space to manage your certificates.
2. **Create an issuer**: Create a Root CA or Intermediate CA.
3. **Create a certificate template**: Create a template to use for issuing certificates.
4. **Issue a certificate**: Use the template to issue a physical certificate.

Issued certificates can be downloaded in PEM format and applied to web servers, API servers, applications, and more.

## Use cases

### Enhance internal system security

* Allow you to issue TLS/SSL certificates to web servers, API servers, databases, and more within your organization to encrypt communications.
* Save money by not having to use public certificates for your internal infrastructure.

### Cross-microservice authentication

* Allow you to issue certificates to be used for mutual authentication between services (mTLS) in your microservice architecture.
* Allow you to implement secure communication in a Service Mesh environment.

### Authenticate IoT devices

* Allow you to issue unique certificates to IoT devices to enhance device authentication and communication security.
* Automatically deploy and manage certificates to lots of devices.

### Development and test environments

* Allow you to reproduce the same certificate structure in development and test environments as in production.
* Establish a secure test environment to proactively identify security vulnerabilities.

### Code signing and document signing

* When you deploy software, you can issue code signing certificates to ensure the integrity of your software.
* Allow you to apply a digital signature to an electronic document to verify the document's authenticity.

## Getting Started

If you're new to Private CA, you can refer to the following guide:

* [Console User Guide](./console-guide.md): Guide to creating and managing stores, issuers, certificate templates, and certificates in the Private CA console.
* [Certificate renewal with ACME](./acme-guide.md): Guide to using Certbot to automatically issue and renew certificates.
* [API v2.0 Guide](./api-guide-v2.0.md): Guide to downloading certificates and retrieving CRLs and OCSPs via the API.

!!! tip "Notice"
   - Private CAs are optimized for issuing certificates for internal use within an organization. If you need a public certificate, you must use an authorized certificate authority.
   - To use the issued certificate, you must enroll the CA chain as a trusted certificate on the client system.
   - The ACME protocol allows you to fully automate certificate issuance and renewal, which can significantly reduce your operational burden.

!!! danger "Caution"
    - Certificate revocation is an irreversible action. You need to decide carefully.
   - You should enable CRLs and OCSP to allow clients to verify revoked certificates.

