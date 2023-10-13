# SSL Certificates

An X.509 certificate is a digital certificate based on the widely accepted International Telecommunications Union (ITU) X.509 standard, which defines the format of public key infrastructure (PKI) certificates. They are used to manage identity and security in internet communications and computer networking.

## Generate a self-signed SSL certificate

### Generate a Certification Authority (CA)

1. Generate a 4096 bit RSA key

    > Enter a strong PEM pass phrase and save in a secure place.

    ```powershell
    openssl genrsa -aes256 -out ca-key.pem 4096
    ```

2. Generate a public CA certificate

    > You should specify the desire **-days** for expiration.

    > When you run the command enter the information that will be incorporated into your certificate request.

    ```powershell
    openssl req -new -x509 -sha256 -days 365 -key ca-key.pem -out ca.pem
    ```

3. This step is optional if you want to view the certificate's content.

    ```powershell
    openssl x509 -in ca.pem -text
    ```

### Generate a Certificate

1. Generate a 4096 bit RSA key

    > In this case we do not need to protect it with a pass phrase.

    ```powershell
    openssl genrsa -out cert-key.pem 4096
    ```

2. Create a Certificate Signing Request (CSR)

    > You can find the subject in the certificate's content.

    ```powershell
    openssl req -new -sha256 -subj "/CN=yourcn_or_your_name" -key cert-key.pem -out cert.csr
    ```

3. Create a file `openssl.config` with all the alternative names and the basic configuration

        basicConstraints       = CA:FALSE
        authorityKeyIdentifier = keyid:always, issuer:always
        keyUsage               = nonRepudiation, digitalSignature, keyEncipherment, dataEncipherment
        subjectAltName         = @alt_names

        [ alt_names ]
        DNS.1                  = test.local
        DNS.2                  = *.test.local
        IP.1                   = your-ipv4-server  # optional
        IP.1                   = your-ipv6-server  # optional
    
4. Create the certificate

    > You should specify the desire **-days** for expiration.

    > You’ll need to supply the CA’s passphrase to complete the process.

    ```powershell
    openssl x509 -req -sha256 -days 365 -in cert.csr -CA ca.pem -CAkey ca-key.pem -out cert.pem -extfile openssl.cnf -CAcreateserial
    ```

5. Verify the certificates
    ```powershell
    openssl verify -CAfile ca.pem -verbose cert.pem
    ```

6. Create fullchain.pem from cert.pem and ca.pem
   
    ```powershell
    cat path_to_cert.pem > fullchain.pem
    cat path_to_ca.pem >> .\fullchain.pem
    ```

### Install certificate in NGINX

- Edit NGINX configuration file

    ```nginx
    server {
        listen              443 ssl http2;
        listen              [::]:443 ssl http2;
        server_name         domain.local;

        ssl_certificate     path_to_fullchain.pem;
        ssl_certificate_key path_to_cert-key.pem;
        
        # your extra configuration...
    }
    ```

- Restart the NGINX server

### Install the CA Cert as a trusted root CA

#### Windows

-   Using PowerShell you can use this command (with admin permission):

    ```powershell
    Import-Certificate -FilePath "path_to_ca.pem" -CertStoreLocation Cert:\LocalMachine\Root
    ```

    -   In case you want to trust certificates only for the logged in user set
        `-CertStoreLocation` to `Cert:\CurrentUser\Root` .

-   Using Command Prompt you can use this command:

    ```powershell
    certutil.exe -addstore root C:\ca.pem
    ```