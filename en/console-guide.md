# Console User Guide
**Management > Private CA > Console User Guide**

Private CA console is organized around a certificate authority (CA), and all resources (certificate templates, issuers, certificates, ACME tokens) belong to a specific repository. The console screen is tabbed, with a list of repositories on the left and details about the selected repository on the right.

## Private CA usage flow

The process of getting a certificate from a private CA is as follows:

1. **Create a repository**: create a space to manage your certificates.
2. **Create issuer**: create a certificate authority (CA) to sign the certificate.
    - Root CA: top-level certificate authorities
    - Intermediate CA: intermediate certificate authorities under the Root CA
3. **Create a certificate template**: use to issue multiple certificates with the same configuration.
4. **Issue a certificate**: issue the actual certificate using the certificate template.

!!! tip "Notice"
    - **certificate authority (CA)**: the entity that issues and signs certificates.
    - **Root CA**: a self-signed top-level certificate. The starting point for all trust.
    - **Intermediate CA**: an intermediate certificate signed by the Root CA. used to issue the actual server certificate.

## Repository

A repository is the basic unit for managing a private CA. Once you create a repository, you can manage issuers, certificate templates, certificates, and more.

### Add repository

1. Click **+ Add** in the top left corner of the console to add a repository.
  ![ca_empty_list](https://static.toastoven.net/prod_privateca/2025-12-23_ko/ca_init.png)

2. In the Add Repository modal window, enter the following information:
  ![ ca_create](https://static.toastoven.net/prod_privateca/2025-12-23_ko/ca_create.png)
    - **Repository name** (required): Enter a name to identify the repository.
    - **Repository description** (optional): enter a description for the repository.
    - **Enable CRL**
        - Select whether to enable a certificate revocation list (CRL).
        - Provide a list of revoked certificates periodically so clients can check certificate validity.
        - If you enable CRLs, you can set the renewal interval to every day.
    - **Enable OCSP**
        - Select whether to enable online certificate status protocol (OCSP).
        - A protocol that allows you to quickly check the revocation status of individual certificates to their status at the time of the request.
        - When OCSP is enabled, you can set the renewal interval in hours.

3. Click **Create** to create the repository.

### Modify and delete repositories

In the repository list, you can click the menu button (⋮) to the right of each repository entry to perform the following actions:
![overview\_3dot](https://static.toastoven.net/prod_privateca/2025-12-23_ko/overview_3dot.png)

- **Modify**: you can change the repository's name, description, and CRL/OCSP settings.
- **Delete**: delete the repository.
    - When you delete a repository, all resources that belong to it (issuers, certificate templates, certificates, and ACME tokens) are deleted along with it.

!!! danger "Caution"
    The delete operation is irreversible, so use caution.

### Repository details

Click the target repository in the list of repositories on the left, and you'll see the details of the repository on the right. The repository details screen consists of the repository name, a description, a list of tabs, and other details.

#### Tab list

When you select a repository, you'll see the following tabs at the top of the screen on the right, each of which you can click to jump to the corresponding feature:
![overview_tabs](https://static.toastoven.net/prod_privateca/2025-12-23_ko/overview_tabs.png)

- **Overview**: statistics and settings information for your repository
- **Certificate template**: list and manage certificate templates
- **Issuer**: list and manage certificate issuers
- **Certificate**: list and manage issued certificates
- **ACME management**: list and manage ACME tokens
- **Certificate history**: check the certificate history of a repository

#### Resource statistics card

At the top of the screen, you'll see three cards that show the number of key resources in your repository.
![overview_resource_card](https://static.toastoven.net/prod_privateca/2025-12-23_ko/overview_resource_card.png)

- **Certificate template**: Total number of certificate templates created
- **Issuer**: Total number of created issuers (Root CA, Intermediate CA)
- **Certificate**: Total number of issued certificates

Clicking **View {card name} >** on each card will take you directly to the Manage tab for the resource.

#### ACME info

The bottom of the resource card displays ACME information:
![overview_acme_info](https://static.toastoven.net/prod_privateca/2025-12-23_ko/overview_acme_info.png)

- **Full token**: Total number of ACME tokens created
- **Active token**: Number of active ACME tokens
- **Deleted tokens**: Number of ACME tokens deleted

#### Repository details

At the bottom of the ACME information, you'll see the repository details.
![overview_detail](https://static.toastoven.net/prod_privateca/2025-12-23_ko/overview_detail.png)

- **Repository ID**: ID of the repository
- **CRL URL**: URL to view the certificate revocation list
- **CRL renewal cycle**: how often the CRL is renewed (in days)
- **OCSP URL**: online certificate status protocol (OCSP) responder URL
- **OCSP renewal cycle**: how often OCSP information is updated (in hours)

!!! tip "Notice"
    CRLs and OCSPs are ways to verify a certificate's revocation status. The CRL provides a list of revoked certificates, and OCSPs can quickly look up the status of individual certificates to the status at the time of the request.

## Issuer

Issuers are the certificate authorities that sign and issue certificates. In Private CA, you can create two types of issuers: Root CA and Intermediate CA.

### Guide to selecting an issuer type

- **If you are using only the Root CA**: issue certificates for internal use in small organizations
- **When using Root CA + Intermediate CA**
    - When you want to keep the Root CA's private key secure
    - When you want to run separate CAs for different departments/projects
    - When you want to follow security best practices (recommended)

### Issuer list

On the Issuer tab, you can see all of your created issuers in a table. The table displays the following information:
![issuer\_list\_after](https://static.toastoven.net/prod_privateca/2025-12-23_ko/issuer_list_after.png)

- **Name**: Issuer's name
- **Status**: Issuer's current status
    - **active**: Normally available (blue)
    - **revoked**: Revoked (red)
- **Type**: Root or Intermediate
- **Serial number**: The certificate's unique serial number
- **Common name**: common name of the certificate

Each issuer entry has **Revoke** button so that you can revoke the issuer when needed.

### Add an issuer

1. On the Issuer tab, click **+ Add**.
![issuer\_list](https://static.toastoven.net/prod_privateca/2025-12-23_ko/issuer_list.png)


2. On the Create Issuer page, enter the following information:
![issuer_create](https://static.toastoven.net/prod_privateca/2025-12-23_ko/issuer_create.png)
    - Basic Info
        - **Issuer type**: select Root or Intermediate as the issuer's type
            - **Root**: a top-level certificate authority, which is a self-signed certificate.
            - **Intermediate**: an intermediate certificate authority, signed by the Root CA.
                - **Parent certificate ID**: if you selected the Intermediate type, select a parent issuer.
                  ![issuer_create_intermediate](https://static.toastoven.net/prod_privateca/2025-12-23_ko/issuer_create_intermediate.png)
        - **Issuer name** (required): A name to identify the issuer
        - **Issuer description** (optional): Description of the issuer
        - **Common name** (required): Certificate common name
        - **Expiration setting** (required): Enter an expiration date, choose TTL or specific date
            - **TTL**: Valid for a specified period of time from the time of issuance (for example: 365d, 8760h, 60m, 30s)
            - **Specific date**: Specify a specific expiration date (not after)
        - **Backdate Validity**: This period allows the certificate's validity start time to be set earlier than the current time. Used to prevent time synchronization issues (default: 30s / e.g. 1d, 24h, 60m, and 30s)
        - **Maximum path length**: specify the maximum number of intermediate CAs allowed under this issuer in the certificate chain. A value of 0 means that no more subordinate CAs can be created (e.g: 0)

    - Key Info
        - **Key algorithm**: Choose among RSA, EC, and ED25519
        - **Key bit**: algorithmic key bit selection

    - Subject alternative name (SAN) configuration
        - **Exclude common names from SANs**: select whether to automatically exclude common names (CNs) from the SAN list.
        - **Subject serial number**: enter the subject's unique serial number.
        - **Subject alternate names (SAN**): Additional distinguished name in domain format (e.g., example.com, sub.example.com)
        - **IP subject alternate names (IP SANs**): an additional identifying name in the form of an IP address (e.g. 192.168.1.1, 10.0.0.1)
        - **URI subject alternate names (URI SANs**): additional identifying name in URI format (e.g., https://example.com, spiffe://example.org)
        - **Other SANs**: other types of SANs (e.g: 1.2.3.4;UTF8:test@example.com

    - Subject information (Subject)
        - **Country (C)**: country code
        - **State/Town (ST)**: state or province
        - **State, county, or district (L)**: city name
        - **Road name address**: road name address
        - **Postal code**: postal code
        - **Organization (O)**: Organization name
        - **Department (organizational unit) (OU**): department name

3. Click **Add** to add the issuer.

### Issuer details

Click the issuer's name in the issuer list to go to the details page. The details page displays the following information, and you can download the certificate PEM file via the Download button at the top.
![issuer\_detail](https://static.toastoven.net/prod_privateca/2025-12-23_ko/issuer_detail.png)

#### Certificate information
- Status, type, serial number
- Subject information (subject DN)
- Issuer information (issuer DN)
- Key usage and extended key usage
- Algorithms and key sizes
- Validity period (not before, not after)
- Certificate PEM contents

#### Issuer URL
- **Issued certificate URL**: list of certificates issued by this issuer
- **CRL distribution point**: URL to verify the CRL
- **OCSP server**: OCSP responder URL


### Issuer modification, revocation

#### Modify issuer
You can modify the name and description directly on the issuer details page. After making your edits, click **Save** to save your changes.

- Editable fields
    - **Name**: you can modify the issuer name.
    - **Description**: you can modify the issuer description.

#### Issuer revocation
1. In the Issuers list, click **Revoke** for the issuer you want to revoke.
2. In the confirmation dialog box, click **Revoke**to confirm the revoke.

!!! danger "Caution"
    - Revoking an issuer affects the trustworthiness of all certificates issued by that issuer. A revoked issuer can no longer issue certificates, and already issued certificates can be checked for revocation status via CRL or OCSP.
    - Root certificates cannot be revoked.

## Certificate template

A certificate template is a collection of settings for issuing certificates quickly and consistently. Certificate templates make it easy to issue multiple certificates with the same configuration.

### List of certificate templates

On the Certificate Template tab, you can see all the certificate templates that have been created in a table. The table displays the following information:
![template_list_after](https://static.toastoven.net/prod_privateca/2025-12-23_ko/template_list_after.png)

- **Name**: Click the certificate template name to go to the details.
- **Description**: Description of certificate templates

Each certificate template entry has **Modify** and **Delete** buttons to help you manage your certificate templates.

### Add a certificate template

1. On the Certificate Template tab, click **+ Add**.
![template_list](https://static.toastoven.net/prod_privateca/2025-12-23_ko/template_list.png)

2. On the Create Certificate Template page, enter the following information:
![template_create](https://static.toastoven.net/prod_privateca/2025-12-23_ko/template_create.png)

    - Basic Info
        - **Certificate template name** (required): a name to identify the certificate template
        - **Description** (optional): description of certificate templates
        - **Select issuer**: Select an issuer to sign the certificate created with this certificate template.

    - Limit settings
        - **Expiration settings** (required)
            - **TTL**: set a maximum validity period (e.g: 365d, 8760h, 60m, 30s)
            - **Specific date**: specify a fixed expiration date (not after)
        - **Backdate Validity**: This period allows the certificate's validity start time to be set earlier than the current time. Used to prevent time synchronization issues (default: 30s / e.g. 1d, 24h, 60m, and 30s)

    - SAN option
        - **Allow IP SANs**: allow IP addresses to be included in the SAN.
        - **URI subject alternate names (URI SANs**): enter the SAN in URI format (e.g. https://example.com, spiffe://example.org).
        - **Other SANs**: enter other types of SANs (e.g. 1.2.3.4;UTF8:test@example.com

    - Common applied settings
        - Settings
            - **Server-side storage option**: select whether you want to store the created certificate on the server.
            - **Enable basic constraints for non-CA**: Select whether to specify in the certificate that you are not a CA.

        - Key parameters
            - **Key algorithm**: Choose among RSA, EC, and ED25519
            - **Key bit**: algorithmic key bit selection
            - **Signature bit**: Select the number of bits in the hash algorithm to use for signing the certificate

            !!! danger "Caution"
                Signature bits can only be set when using the RSA algorithm. Otherwise, it is ignored by the algorithm.

        - Key usage
            - Select the purpose of the certificate: `digitalSignature`, `keyEncipherment`, `keyCertSign`, or certificate signing.

        - Extended key usage
            - Select what you want to use the extended key for: `serverAuth`(TLS server authentication), `clientAuth`(TLS client authentication), `codeSigning, or codeSigning`.
            - **Extended key usage OIDs**: you can manually enter an OID for additional extended key purposes (e.g. 1.3.6.1.5.5.7.3.1, 1.3.6.1.5.5.7.3.2)

        - Certificate policies
            - **List of policies**: enter an OID that represents the policy the certificate complies with. you can enter multiple OIDs.
                - Example: 2.5.29.32.0 (anyPolicy), 1.2.3.4.5 (organization-specific policies)
            - The Certificate Policy field specifies under which policy the certificate was issued, and is used to verify compliance with the policy during certificate validation.

    - Additional subject fields
        - **Use the CSR common name**: Select whether to use the CN from the CSR as is for the certificate.
        - **Use CSR SANs**: select whether to include the SAN of the CSR in the certificate.
        - **Country (C)**: country code
        - **State/Town (ST)**: state or province
        - **State, county, or district (L)**: city name
        - **Road name address**: road name address
        - **Postal code**: postal code
        - **Organization (O)**: Organization name
        - **Department (organizational unit) (OU**): department name

        !!! danger "Caution"
            Even if you set a value for the Subject DN in the CSR, it will be overwritten by the value you set in the certificate template.

3. Click **Add** to add a certificate template.

### Certificate template details

Click the certificate template name in the list of certificate templates to go to the details page. The detail page is organized into collapsible sections, where you can see the information entered by the user.
![template_detail](https://static.toastoven.net/prod_privateca/2025-12-23_ko/template_detail.png)

At the top of the details page are the **+ Create New Certificate**, Modify**, and **Delete** ** Certificate** buttons.

### Modify, delete certificate template

#### Modify certificate template
1. In the certificate template list, click **Modify**, or on the details page, click **Modify**.
2. On the Modify Certificate Template page, make the necessary changes.
3. Click **Modify** to save your changes.

#### Delete a certificate template
1. In the list of certificate templates, click **Delete** for the certificate template you want to delete, or on the details page, click **Delete**.
2. In the confirmation dialog box, click **Delete** to confirm the deletion.

!!! tip "Notice"
    Deleting a certificate template does not affect certificates that have already been generated with that certificate template.

### Create certificates with certificate templates

To create a certificate using a certificate template, follow these steps:

1. At the top of the certificate template detail page, click **+ Create New Certificate**.
  ![template\_detail\_generate](https://static.toastoven.net/prod_privateca/2025-12-23_ko/template_detail_generate.png)

2. Select the type of certificate generation.
  ![template\_generate](https://static.toastoven.net/prod_privateca/2025-12-23_ko/template_generate.png)
    - If you select **Certificate CSR signature**, a different form of input appears, as follows:
  ![template\_generate\_csr](https://static.toastoven.net/prod_privateca/2025-12-23_ko/template_generate_csr.png)

3. On the Create a Certificate page, enter the following information:
    - **Common name** (required): subject name of the certificate
    - **Expiration setting** (required): set within the maximum range of the certificate template
    - **SAN information**: additional SAN information

4. Click **OK** to create the certificate.

The generated certificate can be saved to the Private CA at your option, and if so, you can view it on the Certificate tab.

## Certificate

The Certificate tab allows you to view and manage all certificates issued in your repository.

### List of certificates

The Certificate tab shows all issued certificates in a table. The table displays the following information:
![certificate\_list](https://static.toastoven.net/prod_privateca/2025-12-23_ko/certificate_list.png)

- **Common name**: Click the common name of the certificate to go to the details.
- **Status**: Current status of the certificate
    - **active**: Normally available (blue)
    - **revoked**: Revoked (red)
- **Serial number**: The certificate's unique serial number
- **Not before**: when the certificate became valid

Each certificate entry has **Download** and **Revoke** buttons to help you manage your certificates.

### Certificate details

Click the common name in the certificate list to go to the details page. The detail page displays the following information, and you can download the certificate PEM file via the Download button at the top.
![certificate\_detail](https://static.toastoven.net/prod_privateca/2025-12-23_ko/certificate_detail.png)

#### Certificate information
- **Common name**: common name of the certificate
- **Serial number**: unique serial number
- **Certificate**: certificate PEM Information
- **CA chain**: chain certificate PEM information
- **Valid period**
    - **Not before**: when the certificate becomes valid
    - **Not after**: when certificates expire
- **Algorithm and key size**: signing algorithms and key lengths
- **Key usage**: digitalSignature, keyEncipherment, etc.
- **Extended key usage**: serverAuth, clientAuth, etc.

### Revoke certificate

To revoke a certificate, proceed as follows

1. In the list of certificates, click **Revoke**for the certificate template you want to revoke, or on the details page, click **Revoke**.
2. In the confirmation dialog box, click **Revoke**to confirm the revoke.

Revoked certificates are considered no longer trusted, and you can check their revocation status in the following ways:

- **Certificate revocation list (CRL)**: you can see a list of revoked certificates via the CRL URL of the repository.
- **Online certificate status protocol (OCSP)**: you can look up the status of individual certificates via the OCSP URL of the repository.

!!! danger "Caution"
    Certificate revocation is an irreversible action. You can't reactivate a revoked certificate, so you'll need to issue a new one.

## ACME management

An automated certificate management environment (ACME) is a protocol that automates certificate issuance and renewal. The ACME management feature of Private CA allows you to automatically issue certificates through an ACME client, such as the Let's Encrypt client (e.g., certbot).

### ACME token list

On the ACME Management tab, you can view all generated ACME tokens in a table. The table displays the following information:
![acme\_list\_after](https://static.toastoven.net/prod_privateca/2025-12-23_ko/acme_list_after.png)

- **Name**: Click the name of an ACME token to go to its details.
- **ID**: ACME token ID
- **Description**: description of the ACME token

Each token entry has **Delete** button, so you can delete tokens that you no longer use.

### Add an ACME token

1. On the ACME Management tab, click **+ Add ACME Token**.
![acme\_list](https://static.toastoven.net/prod_privateca/2025-12-23_ko/acme_list.png)

2. In the Create ACME token modal window, enter the following information:
![acme\_create](https://static.toastoven.net/prod_privateca/2025-12-23_ko/acme_create.png)
    - **Name** (required): a name to identify the ACME token
    - **Description** (optional): description of the ACME token

3. Click **create** to create the token.

#### Verify information after ACME token is created
![acme\_once](https://static.toastoven.net/prod_privateca/2025-12-23_ko/acme_once.png)
When the token is created, the following information is displayed:

- **Token ID**: identifiers used for ACME client setup
- **HMAC key**: secret key used for ACME client authentication

!!! danger "Caution"
    The HMAC key is only displayed once at token generation. Be sure to copy and store it somewhere safe, or you won't be able to see it again. If you lose your HMAC key, you'll need to generate a new token.

### ACME token details

![acme\_detail](https://static.toastoven.net/prod_privateca/2025-12-23_ko/acme_detail.png)
Click the token name in the token list to go to the details page. The details page displays the following information:

#### Issued certificate
A list of certificates issued using the token is displayed. Each certificate contains the following information:

- **Common name**: Certificate common name
- **Status**: certificate status
- **Serial number**: Certificate serial number
- **Effective date**: Validity start date

### Example of ACME client setup

Use the [Certificate Renewal with ACME](./acme-guide.md) page as a guide to complete it.

### Delete an ACME token

1. On the ACME Management tab, click **Delete** for the token you want to delete.
  ![acme\_detail\_delete](https://static.toastoven.net/prod_privateca/2025-12-23_ko/acme_detail_delete.png)

2. In the confirmation dialog box, click **Delete** to confirm the deletion.

!!! tip "Notice"
    Deleting an ACME token does not affect certificates already issued with that token. However, automatic renewal using that token will no longer work, so you'll need to update your ACME client settings to generate a new token.

## Certificate history

![history](https://static.toastoven.net/prod_privateca/2025-12-23_ko/history.png)
The Certificate History tab provides a chronological view of certificate-related activity that has occurred in the repository. The history includes the following information:

- Issuer, certificate generation history
- Certificate revocation history

Historical information helps you track and audit certificate management activity in your repository.