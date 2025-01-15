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
