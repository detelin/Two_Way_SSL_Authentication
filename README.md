# Two_Way_SSL_Authentication / Client-side SSL
Two-way authentication creates a truststore and a keystore on both the client and the server.

## Using self-signed certificate.
### Create a Certificate Authority root (which represents this server)
Organization & Common Name: Some human identifier for this server CA.

    openssl genrsa -des3 -out ca.key 4096
    openssl req -new -x509 -days 365 -key ca.key -out ca.crt


### Create the Client Key and CSR
Organization & Common Name = Person name

    openssl genrsa -des3 -out client.key 4096
    openssl req -new -key client.key -out client.csr
    # self-signed
    openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt

#### Convert Client Key to PKCS
So that it may be installed in most browsers.

    openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12

#### Convert Client Key to (combined) PEM
Combines `client.crt` and `client.key` into a single PEM file for programs using openssl.

    openssl pkcs12 -in client.p12 -out client.pem -clcerts

### Install Client Key on client device (OS or browser)
Use `client.p12`. Actual instructions vary.

### Install CA cert on nginx
So that the Web server knows to ask for (and validate) a user's Client Key
against the internal CA certificate.

    ssl_client_certificate /path/to/ca.crt;
    ssl_verify_client optional; # or `on` if you require client key

Configure nginx to pass the authentication data to the backend application:

* [Client Side Certificate Auth in Nginx](http://blog.nategood.com/client-side-certificate-authentication-in-ngi), section “Passing to PHP.”
* [SSL module documentation](http://wiki.nginx.org/NginxHttpSslModule)


## Using CACert Keys
* Get [client key from CACert](https://www.cacert.org/account.php?id=36)
* Install client key in client device.
* Install [CACert root certs](https://www.cacert.org/index.php?id=3) in server and client device.
* Configure nginx, as above.

## Information source: https://gist.github.com/mtigas/952344

