# Release notes

**Management > Private CA > Release Notes**

## January 27, 2026

### Feature Updates
- Limited the maximum backdate period to 30 days.
- Updated the label "Expiration Settings" to "Certificate Lifetime" within Templates > Expiration Settings.
- Improved the clarity of error messages triggered by unauthorized user actions.

### Bug Fixes
- Fixed an issue where the expiration method was incorrectly displayed as "TTL" when set to a "Specific Date" in templates.
- Fixed a bug where "TTL" appeared instead of the selected "Specific Date" in Template expiration settings.

## December 23, 2025

### New service launch

* We've launched Private CA service. You can use the Private CA service to create Root CAs and Intermediate CAs to issue certificates for internal use within your organization. Support for the ACME protocol allows you to automatically issue and renew certificates through clients like Certbot. You can also check the status of certificate revocation via the certificate revocation list (CRL) and online certificate status protocol (OCSP).
