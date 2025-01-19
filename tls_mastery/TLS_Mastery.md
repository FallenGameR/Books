# TLS Mastery (by Michael W Lucas)

## 0. Introduction

```bash
openssl s_client -showcerts -connect www.mwl.io:443 < /dev/null | 
  openssl x509 -text -noout
```

- Connect to a web server and get it's cert, do not communicate further
- Parse x509 certificate to a readable format

## 2. TLS Connections

```bash
# These should not work
openssl s_client -brief -ssl3 -crlf www.mwl.io:443
openssl s_client -brief -tls1 -crlf www.mwl.io:443
openssl s_client -brief -tls1_1 -crlf www.mwl.io:443

# These should work - unless TLS 1.2 was already deprecated
openssl s_client -brief -tls1_2 -crlf www.mwl.io:443
openssl s_client -brief -tls1_3 -crlf www.mwl.io:443
```

- `-brief` skips details of TLS connection and gives a short text after which it's possible to work with the server
- `-crlf` is alternative to `-connect` and it differs in a way how CL and LF are treated, for interactive use of web servers `-crlf` is the right switch. Unless your distro swapped it with `-connect`.

## 3. Certificates

DER, PEM, CRT are different certificate encodings (like ASCII and UTF8), one can convert from one to another.

- DER (Distinguished Encoding Rules) is a short binary encoding, common for the Certificate Revocation Lists.
- PEM (Privacy-Enhanced Mail) is a text encoding, can be used to setup certs via web UI. It is base64 encoded DER with header and footer that says that is inside. Header is more trustworthy than file extension.

```bash
# To read a certificate using particular encoding
openssl x509 -in cert_file -inform der -text -noout

# To read a PEM certificate input-format can be ommited
openssl x509 -in cert_file -text -noout

```

- `-inform` is the incoming format that specifies in encoding the cert_file uses. **File extensions may be misleading here**.
- `-text` means to provide human-readable output (don't imply comprehension).
- `-noout` means prevent displaying of the whole certificate

```bash
# To convert between encodings, PEM is default input format
openssl x509 -in cert_file.der -inform der -outform pem -out cert_file.pem
openssl x509 -in cert_file.pem -outform der -out cert_file.der

# When no input is specified STDIN is used, possible to just copy-paste PEM
openssl x509 -outform der -out cert_file.der
```

PKCS (Public Key Cryptography Standards) from RSA laboratories is a set of standards. PKCS12 says how to store multiple encryption files in a single `.p12` or `.pfx` archive. Used a lot in Java that uses several not quite compatible assumptions.

```bash
# Create pkcs archive with specified contents, it would ask to create a password for it
openssl pkcs12 -export -out site.p12 -inkey privkey.pem -in cert.pem -certifile chain.pem

# View pkcs file
openssl pkcs12 -info -in site.p12
```

- `-export` create PKCS archive file
- `-inkey` private key
- `-in` certificate file
- `-certfile` additional certificates (that form the chain of trust to a Certificate Authority). Without the whole chain a certificate can not be validated.
