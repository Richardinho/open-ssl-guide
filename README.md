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

### mandatory options for default_CA section
new_certs_dir
database
certificate
private_key = $dir/private/my-ca-key.pem
default_md
serial
policy

# Options
## new_certs_dir
Specifies the directory where new certificates will be placed.
The generated certificate will be written to this directory with a filename consisting of the serial number and a suffix of `.pem`.
e.g. `my-ca-directory/newcerts/654.pem` (check this is so)
### Syntax
```
  new_certs_dir = <directory>
```
### Examples
```
  # dir is a variable defined elsewhere
  new_certs_dir  = $dir/newcerts
```
*mandatory* : yes
*command line option* : -outdir

I don't know why the command line option is different!


## Config file
*mandatory* means that the options must be in the config file or specified on the command line.

```
# configuration for when `ca` command uses this file to sign certificates

[ ca ]

# The only property that should be in this section. It points to the section containing the default `ca` config.
# command line option: -name
default_ca = CA_default

# the config section for ca. Why this isn't just in the ca section above is a mystery!
[ CA_default ]

# the config file format allows declaration of variables that can be used elswhere in the file (check exact scope)
# check exactly how these work
dir = <file-path>
foo = bar
database_file = $dir/database/index.txt

# MANDATORY OPTIONS

# Specifies the directory where new certificates will be placed.
# command line option: -outdir (I don't know why the difference in names!)
# The certificate will be written to this directory using a filename of the serial number with .pem attached
new_certs_dir = <directory>

# File to use as a database.
# command line option: curiously there isn't one!
database = <file-path>

# The CA certificate. Not (I think) where certificates are created (new_certs_dir).
# command line option: -cert
certificate = <file-path>

# File containing CA private key
# command line option: -keyfile
private_key = <file-path>

# The message digest to use
# command line option: -md
default_md = md5 | sha1 | mdc2

# File containing next number (in hex) to use as a serial number. (Get exact format)
serial = <file-path>
# command line option: none

policy

# NON-MANDATORY

```




