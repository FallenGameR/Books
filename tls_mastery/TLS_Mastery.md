# TLS Mastery (by Michael W Lucas)

## 0. Introduction

```bash
openssl s_client -showcerts -connect www.mwl.io:443 < /dev/null | 
  openssl x509 -text -noout
```

- Connect to a web server and get it's cert, do not communicate further
- Parse x509 certificate to a readable format
