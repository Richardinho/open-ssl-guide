#OpenSSL Usage

I personally find the documentation for OpenSSL to be deeply unsatisfactory, particularly around the config file. In this document, I want to create a guide for usage of OpenSSL, at least as it applies to my usage.



## Use Cases
1. Create a certificate
2. Revoke a certificate



CA: Certificate Authority
Public Key 
CSR: Certificate Signing Request

A CSR is created from a public key and a config file
A certificate is created by a CA (Certificate Authority) from a CSR.


Questions:
When would a client need a certificate?
How is this installed?
How to create/manage serial numbers?


```
# configuration for when `ca` command uses this file to sign certificates

[ ca ]

# The only property that should be in this section. It points to the section containing the default `ca` config.
# command line option: -name
default_ca = CA_default

# the config section for ca. Why this isn't just in the ca section above is a mystery!
[ CA_default ]
  //  options

```

## Options
### new_certs_dir
* *mandatory* : yes
* *command line option* : -outdir

Specifies the directory where new certificates will be placed.
The generated certificate will be written to this directory with a filename consisting of the serial number and a suffix of `.pem`.
e.g. `my-ca-directory/newcerts/654.pem` (check this is so)

#### Syntax
```
  new_certs_dir = <directory>
```

#### Examples
```
  # dir is a variable defined elsewhere
  new_certs_dir  = $dir/newcerts
```

I don't know why the command line option is different!

---

### database
* *mandatory* : yes
* *command line option* : n/a

File to use as a database.

#### Syntax
```
database = <file-path>
```
#### Examples
```
  # dir is a variable defined elsewhere
  database = $dir/index.txt
```
Curiously, there is not command line option for database.

---

### certificate 
* *mandatory* : yes
* *command line option* :  -cert

The CA certificate. Not (I think) where certificates are created (new_certs_dir).

#### Syntax
```
  certificate = <file-path>
```

#### Examples
```
  # dir is a variable defined elsewhere
  certificate = $dir/cacert.pem
```
---

### private_key 
* *mandatory* : yes
* *command line option* : -keyfile

File containing CA private key

#### Syntax

```
  private_key = <file-path>
```

#### Examples

```
  # dir is a variable defined elsewhere
  private_key = $dir/private/my-ca-key.pem
```
---

### default_md 
* *mandatory* : yes
* *command line option* : -md

The message digest to use

#### Syntax
```
  default_md = md5 | sha1 | mdc2
```

#### Examples
```
  default_md     = md5
```
---

### serial 
* *mandatory* : yes
* *command line option* :  n/a

File containing next number (in hex) to use as a serial number. (Get exact format)

#### Syntax
```
  serial = <file-path>
```

#### Examples
```
  # dir is a variable defined elsewhere
  serial = $dir/serial
```
---

### policy 
* *mandatory* : yes
* *command line option* : -policy

Defines the policy of the CA. That is, which fields in the CSR are acceptable 

#### Syntax
```
  policy: <policy>
```

#### Examples
```
  policy = policy_any
```






### xxxxxxx 
* *mandatory* : xxxx
* *command line option* :  xxxxx

xxxxxx

#### Syntax
```
  xxxxx
```

#### Examples
```
  xxxxx
```
