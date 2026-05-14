# [UNIT] Self-Signed-CA

Owner: Nam Tran
Last edited time: April 29, 2026 4:22 PM

- [Terms and Definitions](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
- [Generate custom RootCA (At Root Side)](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Generate RootCA key:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Make SAN (Subject Alternative Name) file for self-signed RootCA:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Self-sign for root Certificate:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Check CRT file:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
- [Create ServerCA (At Servers Side):](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Generate Private Key for Server:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Make SAN file for ServerCSR:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Generate CSR file:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Check CSR file:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
- [Using RootCA to sign ServerCSR (at Root Side)](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Make SAN file for signing Server CSR:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Sign ServerCSR and generate the serial file of root CA:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Check signed X509 Certificate (CRT) file:](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Check matching Server's Private Key - CSR - x509 Certificate files: (at Server side)](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
- [Cheat sheets](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
    - [Converting Using OpenSSL](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)
- [Reference](%5BUNIT%5D%20Self-Signed-CA%201cb675ca799580bc81c3d58d0f5dfec1.md)

# Terms and Definitions

| Abbreviation | Term | Descriptions |
| --- | --- | --- |
| X.509 | X.509 Certificate | Standard format for **public key certificate**. X.509 has been adapted for internet use by the IETF’s Public-Key Infrastructure (X.509) (PKIX) working group.X509 includes+ Subject Name, Issuer Name, Serial Number, Version, Signature Algorithm, Valid Time...+ Public key, Digital signature...+ X.509 v3 includes extensions: Subject Alternative Name, Key Usage... |
| CSR | Certificate Signing Request |  |
| Key | Private key | Cryptographic key |
| PEM | Privacy Enhanced Mail | Base64 ASCII encoding scheme of X.509 Certificate and Key.PEM file is a plain text file containing one or more items, each item has the header and footer (e.g. `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----`)Common extensions: `.crt`, `.pem`, `.cer` for X.509 Certificate; `.key` for private key; `.ca-bundle` for CA Chain |
| DER | Distinguished Encoding Rules | Binary encoding scheme of for X.509 certificates and private keys.Common extensions: `.der`, `.cer` |
| PKCS#7 |  | Container format for digital certificates, is not used to store private keys.Common extensions: `.p7b` |
| PKCS#12 | PKCS12 or PFX | Common binary container format for storing a certificate chain and private key in a single, encryptable file.Common extensions: `.p12`, `.p12` |

# Generate custom RootCA (At Root Side)

## Generate RootCA key:

```jsx
#openssl genrsa -out rootCA_unit.local.key 4096
```

## Make SAN (Subject Alternative Name) file for self-signed RootCA:

```jsx
#nano san_rootCA.conf
[req]
default_bits           = 2048
prompt                 = no
default_md             = sha256
policy                 = policy_match
distinguished_name     = dn
x509_extensions        = x509_ext

[policy_match]
countryName            = optional
stateOrProvinceName    = optional
organizationName       = optional
organizationalUnitName = optional
emailAddress           = optional
commonName             = supplied

[dn]
C                      = VN
ST                     = HCMC
L                      = Ho Chi Minh City
O                      = UNIT TECHNOLOGY CORPORATION
OU                     = Software Development Division
emailAddress           = sysadmin@unit.com.vn
CN                     = unit.local

[x509_ext]
basicConstraints       = critical, CA:true
keyUsage               = critical, digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment, keyAgreement, keyCertSign, cRLSign
extendedKeyUsage       = critical, serverAuth, clientAuth
subjectKeyIdentifier   = hashauthorityKeyIdentifier = keyid:always,issuer
subjectAltName         = @alt_names

[alt_names]
DNS.1                  = unit.local
```

Note:

- `x509_extensions` flag is for self-sign certificate (included in `.crt`). Refer to [https://www.openssl.org/docs/man1.1.1/man5/x509v3_config.html](https://www.openssl.org/docs/man1.1.1/man5/x509v3_config.html)
- Firefox reports SEC_ERROR_CA_CERT_INVALID: need full x509v3 extension described in [https://www.golinuxcloud.com/add-x509-extensions-to-certificate-openssl/](https://www.golinuxcloud.com/add-x509-extensions-to-certificate-openssl/)

## Self-sign for root Certificate:

```jsx
#openssl req -x509 -new -nodes -key rootCA_unit.local.key -out rootCA_unit.local.crt -days 3650 -config san_rootCA.conf
```

## Check CRT file:

```jsx
#openssl x509 -in rootCA_unit.local.crt -noout -text
#openssl x509 -in rootCA_unit.local.crt -noout -text | grep -A15 "X509v3 extensions:"
```

# Create ServerCA (At Servers Side):

![image.png](%5BUNIT%5D%20Self-Signed-CA/image.png)

## Generate Private Key for Server:

```jsx
#openssl genrsa -out wildcard.unit.local.key 2048
```

## Make SAN file for ServerCSR:

```jsx
#nano sanCSR_wildcard.unit.local.conf
[req]
default_bits           = 2048
prompt                 = no
default_md             = sha256
distinguished_name     = dn
req_extensions         = req_ext

[dn]
C                      = VN
ST                     = HCMC
L                      = Ho Chi Minh City
O                      = UNIT TECHNOLOGY CORPORATION
OU                     = Software Development Department
emailAddress           = sysadmin@unit.com.vn
CN                     = *.unit.local

[req_ext]
subjectAltName         = @alt_names

[alt_names]
DNS.1                  = *.unit.local
```

Note: `req_extensions` flag is for Certificate Signing Request (included in `.csr`)

## Generate CSR file:

```jsx
#openssl req -new -key wildcard.unit.local.key -out wildcard.unit.local.csr -config sanCSR_wildcard.unit.local.conf
```

## Check CSR file:

```jsx
#openssl req -in wildcard.unit.local.csr -noout -text
#openssl req -in wildcard.unit.local.csr -noout -text | grep -A10 "X509v3"
```

# Using RootCA to sign ServerCSR (at Root Side)

## Make SAN file for signing Server CSR:

```jsx
#nano sanCA_wildcard.unit.local.conf
basicConstraints       = CA:FALSE
keyUsage               = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage       = serverAuth, clientAuth
authorityKeyIdentifier = keyid, issuer
subjectAltName         = DNS:*.unit.local
```

## Sign ServerCSR and generate the serial file of root CA:

```jsx
#openssl x509 -req -in wildcard.unit.local.csr -CA rootCA_unit.local.crt -CAkey rootCA_unit.local.key -CAcreateserial -out wildcard.unit.local.crt -days 825 -sha256 -extfile sanCA_wildcard.unit.local.conf
```

Note: the expired days (`-days 825`) of ServerCA should be less than 825 days to passed security check of Safari.

## Check signed X509 Certificate (CRT) file:

```jsx
#openssl x509 -in wildcard.unit.local.crt -text -noout
#openssl x509 -in wildcard.unit.local.crt -text -noout | grep -A15 "X509v3 extensions:"
```

## Check matching Server's Private Key - CSR - x509 Certificate files: (at Server side)

For x509 Certificate file:

```bash
#openssl x509 –noout –modulus –in <file>.crt | openssl md5
#openssl x509 –noout -modulus –in wildcard.unit.local.crt | openssl md5
```

For RSA Private key file:

```jsx
#openssl rsa –noout –modulus –in <file>.key | openssl md5
#openssl rsa –noout –modulus –in wildcard.unit.local.key | openssl md5
```

For CSR Request file:

```jsx
#openssl req -noout -modulus -in <file>.csr | openssl md5
#openssl req -noout -modulus -in wildcard.unit.local.csr | openssl md5
```

Three md5 results must equal.

# Cheat sheets

## Converting Using OpenSSL

Convert PEM (end-entity certificate: `.crt` and private key: `.key` and CA chain `.ca-bundle`) to PKCS#12 (`.p12`):

```jsx
#openssl pkcs12 -export -out wildcard.unit.local.p12 -inkey wildcard.unit.local.key -in wildcard.unit.local.crt -certfile rootCA_unit.local.crtEnter
Export Password:
Verifying - Enter Export Password:
```

Convert a PKCS#12 file containing a private key and certificates to PEM:

```jsx
#openssl pkcs12 -in wildcard.unit.local.p12 -out wildcard.unit.local.pem -nodes
Enter Import Password:
MAC verified OK
```

Export a PKCS#12 file (.p12) to `.crt` and private key: `.key` and CA chain `.ca-bundle`:

```jsx
#openssl pkcs12 -in wildcard.unit.local.p12 -nocerts -nodes | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > wildcard1.unit.local.key
Enter Import Password:
MAC verified OK

#openssl pkcs12 -in wildcard.unit.local.p12 -clcerts -nokeys | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > wildcard1.unit.local.crt
Enter Import Password:
MAC verified OK

#openssl pkcs12 -in wildcard.unit.local.p12 -cacerts -nokeys -chain | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > wildcard1.unit.local.ca-bundle
Enter Import Password:
MAC verified OK
```

Convert a PEM Certificate to DER:

```jsx
#openssl x509 -outform der -in wildcard.unit.local.crt -out wildcard.unit.local.der
```

Convert a DER Certificate file to PEM Certificate:

```jsx
#openssl x509 -inform der -in wildcard.unit.local.der -out wildcard2.unit.local.crt
```

# Reference

- [https://www.golinuxcloud.com/tutorial-pki-certificates-authority-ocsp/](https://www.golinuxcloud.com/tutorial-pki-certificates-authority-ocsp/)
- [https://www.ssl.com/how-to/export-certificates-private-key-from-pkcs12-file-with-openssl/](https://www.ssl.com/how-to/export-certificates-private-key-from-pkcs12-file-with-openssl/)
- [https://www.openssl.org/docs/man1.0.2/man1/asn1parse.html](https://www.openssl.org/docs/man1.0.2/man1/asn1parse.html)