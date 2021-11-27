---
title: "How to create ssl cert for tls auth"
date: 2021-10-21
categories:
  - openssl
  - k8s
tags:
  - k8s
  - openssl
---

# Procedures

* create private key
* create csr  (distinguished_name)
* sign csr by CA (x509v3)

# Commands

## Gen private key
```
openssl genrsa \
-out ca.key
```

## Self signed crt
```
openssl req -x509 \
-config openssl.cnf
-new \
-out ca.crt \
-key ca.key
```

## gen csr
```
openssl req \
-config openssl.cnf
-new \
-out server.csr \
-key server.key
```

## sign crt
```
openssl x509 \
-extfile openssl.cnf \
-extensions svr_cert \
-in server.csr \
-out server.crt \
-CA ca.crt \
-CAkey ca.key \
-CAcreateserial
```

* When the CA is first time to sign a cert, you need to create serial file ca.crl with -CAcreateserial
* Next time we sign a new cert, we will need to update the serial file with -CAserial ca.crl

## Outputs

### Inspect the cert / key

```
openssl rsa -in ca.key -noout -text
openssl x509 -in ca.crt -noout -text
```

### extract the pub key

```
openssl rsa -in ca.key -pubout
openssl x509 -in ca.crt -noout -pubkey
```

### Verify

#### crt is issue by ca

```
openssl verify -CAfile ca.crt server.crt
```

#### key and crt are match (modulus)

```
openssl rsa -in ca.key -nout -modulus | openssl md5
openssl x509 -in ca.crt -nout -modulus | openssl md5
```

# trust self signned ca

we need to place the ca.crt to a location and call command below to trusted the cert (the programme will create a soft link)

we can man update-ca-certificate to find the location. eg: /usr/local/share/ca-certificates

```
cp ca.crt /usr/local/share/ca-certificates
update-ca-certificate
```

trusted the cert in firefox and chrome acutally work but we need to restart the pc..

two output should be same

CAxxxx <- Capital in CA, lowercase in next word
-in -out <- represent the module

#  Config

```
cp /etc/ssl/openssl.cnf openssl.cnf
vi openssl.cnf
```

Remove all empty and commended lines and start update the openssl.cnf

```
HOME                    = .
oid_section             = new_oids

[ new_oids ]

tsa_policy1 = 1.2.3.4.1
tsa_policy2 = 1.2.3.4.5.6
tsa_policy3 = 1.2.3.4.5.7

[ ca ]

[ CA_default ]

policy          = policy_match

[ policy_match ]
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_anything ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ req ]
default_bits            = 2048
default_keyfile         = privkey.pem
distinguished_name      = req_distinguished_name
attributes              = req_attributes
string_mask = utf8only

[ req_distinguished_name ]
0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = hhuge9.com
commonName                      = Common Name (e.g. server FQDN or YOUR name)
commonName_max                  = 64

[ req_attributes ]

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment

[ v3_ca ]
basicConstraints = critical,CA:true
keyUsage = critical, keyCertSign

subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer

[ svr_cert ]
basicConstraints=CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth

subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid,issuer

[ usr_cert ]
basicConstraints=CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth

subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid,issuer
```

The most important setting is session: v3_req v3_ca svr_cert and usr_cert

Normally, Cert need req_distinguished_name session only.

For example, we use domain in CN (Common Name) in web server cert

In k8s, we will use CN as username, O (Orgnazation) as group for auth

It can be done by commands without config.

Those information is defined in req stage. (not sign stage)

If you want to use config, you can:
```
[ req_distinguished_name ]
0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = hhuge9.com
commonName                      = Common Name (e.g. server FQDN or YOUR name)
commonName_max                  = 64
```

Or in command, we can

```
openssl req \
-config openssl.cnf
-new \
-out server.csr \
-key server.key \
-subj '/O=hhuge9.com/CN=redis.hhuge9.com'
```

But when we want to create a CA, cert for TLS auth. We need x503v3 extension to define additional information.

That is why we see -config and -extensions in sign command.

To check the documentation of the extension, we can `man openssl x509` > go to bottom > Also see x509v3_config > `man x509v3_config`

CSR do not need the extension. -config is optional.

This three are most important:

For non CA
```
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
```

For CA
```
basicConstraints = CA:TRUE
keyUsage = critical, keyCertSign
```

For server cert
```
extendedKeyUsage = serverAuth
```

For client cert
```
extendedKeyUsage = clientAuth
```

If we want to contraint the cert be used by specfic host, we can add subjectAltName
```
subjectAltName = @alt_name
[ alt_name ]
DNS.0 = redis-master.hhuge9.com
DNS.1 = redis01.hhuge9.com
IP.0 = 192.168.2.100
```

