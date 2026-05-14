# [RHEL] Converting an SSL/TLS certificate from a .crt to a .cert

Owner: Nam Tran
Last edited time: September 19, 2025 5:00 PM

Converting an SSL/TLS certificate from a `.crt` to a `.cert` format often involves changing the file extension or, in some cases, converting the encoding if the target system requires a specific format like DER.

## 1. Renaming the file (for PEM-encoded certificates):

If both `.crt` and `.cert` are intended to represent certificates in the same encoding (most commonly PEM, which is base-64 encoded ASCII text), you can simply rename the file extension.

```bash
mv certificate.crt certificate.cert
```

## 2. Converting encoding using OpenSSL (if DER encoding is required):

If the target system specifically requires the certificate in DER (binary) format, and your `.crt` file is in PEM format, you will need to convert the encoding using OpenSSL.

```bash
openssl x509 -inform PEM -in certificate.crt -outform DER -out certificate.cert
```

### Explanation of the OpenSSL command:

- `openssl x509`: Invokes the OpenSSL utility for X.509 certificate operations.
- `inform PEM`: Specifies that the input file (`certificate.crt`) is in PEM format.
- `in certificate.crt`: Defines the input certificate file.
- `outform DER`: Specifies that the output file (`certificate.cert`) should be in DER format.
- `out certificate.cert`: Defines the output certificate file.

### Important Considerations:

- **PEM vs. DER:** While `.crt` and `.cert` can both refer to PEM-encoded certificates, `.cert` is sometimes used to specifically denote a DER-encoded certificate, particularly in Windows environments.
- **Content verification:** If you are unsure of the encoding, you can open the `.crt` file in a text editor. If it starts with `----BEGIN CERTIFICATE-----`, it is PEM-encoded. If it appears as binary data, it is likely DER-encoded.
- **Private keys:** These instructions are for converting the certificate itself, not the associated private key. Private keys should be handled separately and securely.