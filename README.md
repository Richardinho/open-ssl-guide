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
* When would a client need a certificate?
* How is this installed?
* Is there a way to get Firefox to accept self signed root certificates? 
* What is the difference between x509 v1 and v3?
* How are extensions copied from parent to child certificates?

# Managing serial numbers



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
policy = policy-section 

# non-mandatory
oid_file = ...
oid_section = oid-section 
RANDFILE = ...
default_days = ...
default_startdate = ...
default_enddate = ...
default_crl_hours = ...
default_crl_days = ...
unique_subject = ...
crlnumber = ...
x509_extensions = [x509-extensions] 
crl_extensions = [crl-extensions] 
preserve = ...
email_in_dn = ...
msie_hack = ...
name_opt = ...
cert_opt = ...
copy_extensions = ...

[oid-section]
 ...

[policy-section]
  ...

[x509-extensions]
  ...
 
[crl-extensions]
  ...

```

## Options for CA_default section
### new_certs_dir
* *mandatory* : yes
* *command line option* : `-outdir`

Specifies the directory where new certificates will be placed.
The generated certificate will be written to this directory with a filename consisting of the serial number and a suffix of `.pem`.
e.g. `my-ca-directory/newcerts/654.pem` (check this is so)

[link text](#cookbook)
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
* *command line option* :  `-cert`

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
* *command line option* : `-keyfile`

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
* *command line option* : `-md`

The message digest to use

#### Syntax
```
  default_md = md5 | sha1 | mdc2
```

#### Examples
```
  default_md = md5
```
---

### serial 
* *mandatory* : yes
* *command line option* :  n/a

File containing the next number (in hex) to use as a serial number.
This file can be created with the following command:

```
openssl rand -hex 16  > db/serial
```
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
* *command line option* : `-policy`

Defines the policy of the CA. That is, which DN fields provided in the CSR are acceptable.

#### Syntax
The value of the policy property points to another section in the config file. This section contains 
distinguished name fields which may or may not be present in the CSR for the certificate to be created.
```
  policy: <section-name>

  [<section-name>]

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


### oid_file
* *mandatory* : no
* *command line option* :  n/a

File containing additional object identifiers
What an object identifier is and what it's for, is not so clear to me.
There is a tutorial by [quovadis global](https://support.quovadisglobal.com/kb/a471/inserting-custom-oids-into-openssl.aspx) which gives some explanation.

#### Syntax
```
  oid_file = <file-path>
```

---

### oid_section
* *mandatory* : no
* *command line option* :  n/a

points to section in this file with additional object identifiers

#### Syntax
```
  oid_section = <section-name>
```

---

### RANDFILE 
* *mandatory* : no
* *command line option* :  n/a

File from which to generate random numbers
I don't know the reason for the capitalisation.
The BSD documents don't include this option.

#### Syntax
```
  RANDFILE = <file-path>
```

---

### default_days 
* *mandatory* : no
* *command line option* :  `-days`

Number of days to certify a certificate for

#### Syntax
```
  default_days = <integer>
```

#### Examples
```
  default_days = 365
```
---

### default_startdate 
* *mandatory* : no
* *command line option* :  `-startdate`

The date from which the certificate is effective. I wonder when you'd want to set this for a certificate authority as opposed to for each certificate?


#### Syntax
```
  default_startdate = [YY]YYMMDDHHMMSSZ
```

#### Examples
```
  # January 1st 2008 
  default_startdate = 200801010000Z 
```
---

### default_enddate
* *mandatory* : no
* *command line option* :  `-enddate`

The date on which the certificate will expire. Although not explicitly mandatory, the OpenSSL docs suggest that this must be present.

#### Syntax
```
  default_enddate = [YY]YYMMDDHHMMSSZ
```

---

### default_crl_hours 
* *mandatory* : no (but see below) 
* *command line option* : `-crlhours`

The number of hours before the next CRL is due
Either this or default_crl_days must be present for CRLs

#### Syntax
I suspect the value will be an integer, but the documentation does not say.

---
### default_crl_days
* *mandatory* :  no (but see below)
* *command line option* : `-crldays`

The number of days before the next CRL is due
Either this or default_crl_time must be present for CRLs

#### Syntax
I suspect the value will be an integer, but the documentation does not say.

---

### unique_subject 
* *mandatory* : no
* *command line option* :  n/a

Whether or not certificates should have unique subjects. 

#### Syntax
```
  unique_subject = yes | no
```

---

### crlnumber 
* *mandatory* : no
* *command line option* :  n/a

File containing the next number (in hex) to use for CRL
(Possible related to serial option?)

#### Syntax
```
  crlnumber = <file-path>
```

---

### x509_extensions 
* *mandatory* : no
* *command line option* :  `-extensions`

According to documents: *The section of the configuration file containing certificate extensions to be added when a certificate is issued (defaults to x509_extensions unless the -extfile option is used). If no extension section is present then, a V1 certificate is created. If the extension section is present (even if it is empty), then a V3 certificate is created. See the x509v3_config(5) manual page for details of the extension section format.*
#### Syntax
```
  x509_extensions = <section-name>

```

#### Examples

```

  x509_extensions = foo

  # later in document

  [foo]
  basicConstraints        = critical,CA:true
  keyUsage                = critical,keyCertSign,cRLSign
  subjectKeyIdentifier    = hash

```

I'll discuss these options below.

---

### crl_extensions 
* *mandatory* : no
* *command line option* :  `-crlexts`

Points to section containing CRL extensions. I don't think this is used as much as x509 extensions(?).

#### Syntax
```
  crl_extensions = <section-name>
```

---

### preserve 
* *mandatory* : no
* *command line option* :  `-preserveDN`

According to the documents: *Normally the DN order of a certificate is the same as the order of the fields in the relevant policy section. When this option is set the order is the same as the request. This is largely for compatibility with the older IE enrollment control which would only accept certificates if their DNs match the order of the request. This is not needed for Xenroll.*

#### Syntax
The documents do not say what the type of this value is!

---

### email_in_dn
* *mandatory* : no
* *command line option* :  `-noemailDN`

if *no*, email will be removed from certificate.
The default is just to allow.

#### Syntax
```
  email_in_dn = no
```

---

### msie_hack
* *mandatory* : no
* *command line option* :  `-msie_hack`

Legacy option for dealing with old versions of IE.

---

### name_opt
* *mandatory* : no
* *command line option* :  n/a

Determine how certificate details are displayed to user when requesting signing confirmation.

#### Syntax
```
  name_opt = <name-option> [, <name-option>]

  <name-option> : See documentation
```

#### Examples
```
  name_opt =  ca_default
```
---

### cert_opt
* *mandatory* : no
* *command line option* :  n/a

Determine how certificate details are displayed to user when requesting signing confirmation.

#### Syntax
```
  cert_opt = <name-option> [, <name-option>]
```

#### Examples
```
  cert_opt = ca_default

  <name-option> : See documentation
```
--- 

### copy_extensions
* *mandatory* : no
* *command line option* :  n/a

According to documentation: *Determines how extensions in certificate requests should be handled. If set to none or this option is not present then extensions are ignored and not copied to the certificate. If set to copy then any extensions present in the request that are not already present are copied to the certificate. If set to copyall then all extensions in the request are copied to the certificate: if the extension is already present in the certificate it is deleted first. See the WARNINGS section before using this option. The main use of this option is to allow a certificate request to supply values for certain extensions such as subjectAltName.*

#### Syntax
```
  copy_extensions = none | copy | copyall
```

## req command

The req command is described in the documentation as a "certificate request and certificate generating utility". Close inspection of the the documentation show that several distinct actions can be carried out using this command by specifiying different command line flags.
Here are the command line options grouped according to their function. This grouping is not in the documentation but is my interpretation.
#### Actions
-verify : verifies the signature on the request
-new : generates a new certificate request
-newkey : creates a new certificate request and a new private key so you only need one command instead of two.
-x509 : outputs a self-signed certificate instead of a certificate request

#### debugging
-text : prints out certificate request in text form
-subject : prints out request subject
-pubkey : outputs the public key
-modulus : prints out modulus of the public key
#### configuration

-inform : specifies the input format (DER | PEM)
-outform
-in : input file name for request. I think this means that you can use an existing request as the basis for the new one?
-passin
-out
-passout
-noout
-subj
-rand
-pkeyopt
-key : file to read private key from
-keyform
-keyout : filename to write newly created private key to. Presumably this only works when you use the -newkey option?
-nodes : if a private key is created, it will not be encrypted.
-[digest]
-config
-multivalue-rdn
-days
-set_serial
-extensions
-reqexts
-utf8
-nameopt
-reqopt
-asn1-kludge
-no-asn1-kludge
-newhdr
-batch
-verbose
-engine
-keygen_engine_id

#### input files

## Configuration file for req command

```

[req]
input_password = ...
output_password = ...
default_bits = ...
default_keyfile = ...
oid_file = ..
oid_section = oid-section
RANDFILE = ..
encrypt_key = ...
default_md = ...
string_mask = ...
req_extensions = req-extensions
x509_extensions = x509-sections
prompt = ...
utf8 = ...
attributes = attributes
distinguished_name = distinguished-name

[oid-section]
  ...

[req-extensions]
 ...

[x509-sections]
  ...

[attributes]
  ...

[distinguished-name]
  ...


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
---

input_password = ...
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
---


output_password = ...
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
---


default_bits = ...
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
---

default_keyfile = ...
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
---

oid_file = ..
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
---

oid_section = oid-section
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
---

RANDFILE = ..
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
---

encrypt_key = ...
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
---

default_md = ...
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
---

string_mask = ...
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
---

req_extensions = req-extensions
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
---

x509_extensions = x509-sections
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
---

prompt = ...
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
---

utf8 = ...
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
---

attributes = attributes
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
---

distinguished_name = distinguished-name
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
---

<a name="cookbook">cookbook</a>
