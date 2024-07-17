
# Introduction

OpenSSL is a free and open-source cryptographic library that provides several command-line tools for handling digital certificates. 
Some of these tools can be used to act as a certificate authority.

A certificate authority (CA) is an entity that signs digital certificates. 
Many websites need to let their customers know that the connection is secure, so they pay an internationally 
trusted CA (eg, VeriSign, DigiCert) to sign a certificate for their domain.

In some cases it may make more sense to act as your own CA, rather than paying a CA like DigiCert. 
Common cases include securing an intranet website, or for issuing certificates to clients to allow 
them to authenticate to a server (eg, Apache, Nginx, OpenVPN).



## SCRF Applications
* Stig-manager - Version {{ stigman.version }}
* Keycloak     - Version {{ keycloak.version }}

## Dependencies
* OpenJDK 17
* Nginx
* MySQL

## Changelog
STIG Manager Documentation-0.1.1 (2024-07-05)

* Fixed admonition issues by adding etra_sass plugin
* Updated Mkdocs Bower dependencies to most recent versions
* Changed footer/copyright link to Material theme to GitHub pages
* Made MkDocs building/serving in build process optional

STIG Manager Documentation-0.1.0 (2024-06-17)

* Initial release
