# Release notes
**Management > Private CA > Release Notes**

## March 24, 2026

### Added Features
- Added repository APIs (list, get, create, update, and delete) to the API v2.0 guide.
- Added certificate (issuer) APIs (list, get, issue, and revoke) to the API v2.0 guide.
- Added template APIs (list, get, create, update, delete, and issue certificate) to the API v2.0 guide.
- Added pagination and search query parameters to the repository, certificate, and template list APIs.

### Feature Updates
- Updated the API v2.0 endpoint path from `/cas/{caId}` to `/ca-stores/{caStoreId}`.
- Added detailed constraints (maximum length, value range, default values, etc.) to the request fields in the API v2.0 guide.

## January 27, 2026

### Feature Updates
- Limited the maximum backdate period to 30 days.
- Updated the label "Expiration Settings" to "Certificate Lifetime" within **Templates > Expiration Settings**.
- Improved the clarity of error messages triggered by unauthorized user actions.

### Bug Fixes
- Fixed an issue where the expiration method was incorrectly displayed as "TTL" when set to a "Specific Date" in templates.
- Fixed a bug where "TTL" appeared instead of the selected "Specific Date" in Template expiration settings.

## December 23, 2025

### New service launch
* We've launched Private CA service. You can use the Private CA service to create Root CAs and Intermediate CAs to issue certificates for internal use within your organization. Support for the ACME protocol allows you to automatically issue and renew certificates with clients like Certbot. You can also check the status of certificate revocation with the certificate revocation list (CRL) and online certificate status protocol (OCSP).