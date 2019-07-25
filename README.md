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


# Config file
This is the basic structure of a config file. The ca section denotes options for when the `ca` command is invoked using this as a config.
The only property this section should have is `default_ca` which points to the actual config section. Why it is done this way is unknown to me.

```
[ ca ]

default_ca = CA_default

[ CA_default ]
# mandatory 
new_certs_dir = ...
certificate = ...
private_key = ...
default_md = ...
database = ...
serial = ...
policy = ...

# non-mandatory
oid_file = ...
oid_section = ...
RANDFILE = ...
default_days = ...
default_startdate = ...
default_enddate = ...
default_crl_hours default_crl_days = ...
unique_subject = ...
crlnumber = ...
x509_extensions = ...
crl_extensions = ...
preserve = ...
email_in_dn = ...
msie_hack = ...
name_opt = ...
cert_opt = ...
copy_extensions = ...

```

## Options for CA_default section
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

Defines the policy of the CA. That is, which DN fields provided in the CSR are acceptable.

#### Syntax
The value of the policy property points to another section in the config file. This section contains 
distinguished name fields which may or may not be present in the CSR for the certificate to be created.
```
  policy: <policy-section>

  [<policy-section>]

  <dn-field> : match | supplied | optional

```

The values of this property have the folowing meanings:
* **match** : must be present and have same value 
* **supplied** : must be present
* **optional** :  may be present

If a field is not specified, it is silently deleted unless `-preserveDN` is set.

#### Examples
```
  policy = policy_any

  # later in file

  [ policy_any ]
     countryName            = supplied
     stateOrProvinceName    = optional
     organizationName       = optional
     organizationalUnitName = optional
     commonName             = supplied
     emailAddress           = optional
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
