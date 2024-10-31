1. Create a Certificate Authority
```
openssl genrsa -aes256 -out ca-key.pem 4096
```
2. Create a root-certificate which has to be imported by the clients/browsers o trust the certificates issued/signed by this CA
```
openssl req -x509 -new -nodes -extensions v3_ca -key ca-key.pem -days 365 -out ca-root.pem -sha512
```
3. Create a certificate for the webserver.
```
openssl genrsa -out webserver-key.pem 4096
```
4. Create a CSR. FQDN of the CA’s certificate and the FQDN of the webserver’s certificate have to be different.
```
openssl req -new -key webserver-key.pem -out webserver.csr -sha512
```
5. Create an extfile `webserver.ext` for the webserver’s certificate which contains at least one line for `alt_names` (you can add multiple lines `DNS.2 = ..., DNS.3 = ...` to create a certificate valid for multiple hostnames)
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = www.example.com
DNS.2 = ...
DNS.3 = ...
```
6. Create the public key for the private.key.
```
openssl x509 -req -in webserver.csr -CA ca-root.pem -CAkey ca-key.pem -CAcreateserial -out webserver-pub.pem -days 90 -sha512 --extfile webserver.ext
```
7. Verify that your certificate is signed correctly
```
openssl verify -verbose -CAfile ca-root.pem webserver-pub.pem
```
