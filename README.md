# CA Tools

CA Tools is a set of scripts that creates and manages a self-signed certificate
authority. We use this to issue certificates for our network equipment.

The CA and certificates generated by CA Tools are limited in a number of ways
to mitigate the potential damage if the CA is compromised:

- The CA's basic constraints include a pathlen of 0. This means nobody can
  issue intermediate CAs.
- The CA has a name constraint that is limited to one DNS name or entry. This
  means you can restrict the CA to only issue certificates that are children of
a given domain or subdomain.

Certificates generated by CA Tools always use Subject Alternative Names, even
if the certificate only has one name. This means the certificates will be
trusted by Chrome 58 and newer. All CA Tools certificates use SHA-256.

**WARNING:** Name constraints are not supported in macOS as of version 10.12. If
you mark the CA as trusted for SSL in Keychain Access, Chrome, Safari, and
anything else linked against CommonCrypto (curl, etc.) will validate
certificates that don't match the CA's name constraints even though the name
constraint extension is marked as critical. Firefox, which uses its own trust
store and TLS implementation will properly validate name constraints on macOS.

## Requirements

CA Tools requires Ruby compiled with libyaml, as well as Bash, and the
OpenSSL command line tool.

## Setup

First, create a config.yml file. This contains your organization's info (the
distinguished name).

```yaml
C: US
ST: New York
L: New York
O: ACME Widgets
OU: ACME Widgets IT
emailAddress: acme@example.com
```

Then create the certificate authority:

```
$ ruby mkca.rb "ACME Widgets Certificate Authority" .corp.example.com
```

This will create ca.key and ca.crt. Keep them safe. The second argument to
mkca.rb is the name constraint. The leading dot means that ca.crt will be able
to sign any number of subdomains of corp.example.com, though not
corp.example.com itself. You can find more info in [RFC 5280](https://tools.ietf.org/html/rfc5280#section-4.2.1.10).

Ca.key contains a 4096 bit RSA key pair. Ca.crt is valid for five years. You
must tell your operating system or browser to trust ca.crt in order to trust
any certificates you generate.

## Generating a certificate

Once you have created your CA, you can create a certificate as follows:

```
$ ruby mkcert.rb 4096 switch1.corp.example.com
```

This will generate switch1.corp.example.com.key and
switch1.corp.exmaple.com.crt. The second argument to mkcert.rb is the key size.
You can also specify multiple hosts in one certificate:

```
$ ruby mkcert.rb 4096 switch1.corp.example.com switch2.corp.example.com
```

The first host will be used as the common name for the certificate. It will
also be used for the filenames of the key and the certificate.

## Copyright

Copyright Recurse Center 2017

## License

GPLv3 or later. See COPYING.md for more info.
