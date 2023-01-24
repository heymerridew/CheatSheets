# SSL Certificates

An X.509 certificate is a digital certificate based on the widely accepted International Telecommunications Union (ITU) X.509 standard, which defines the format of public key infrastructure (PKI) certificates. They are used to manage identity and security in internet communications and computer networking.

## Generate a self-signed SSL certificate

### Generate a Certification Authority (CA)

1. Generate a 4096 bit RSA key

    > Enter a strong PEM pass phrase and save in a secure place.

    ```bash
    openssl genrsa -aes256 -out ca-key.pem 4096
    ```

2. Generate a public CA certificate

    > You should specify the desire **-days** for expiration.

    > When you run the command enter the information that will be incorporated into your certificate request.

    ```bash
    openssl req -new -x509 -sha256 -days 365 -key ca-key.pem -out ca.pem
    ```

3. This step is optional if you want to view the certificate's content.

    ```bash
    openssl genrsa -aes256 -out ca-key.pem 4096
    ```

### Generate a Certificate

1. Generate a 4096 bit RSA key

    > In this case we do not need to protect it with a pass phrase.

    ```bash
    openssl genrsa -out cert-key.pem 4096
    ```

2. Create a Certificate Signing Request (CSR)

    > You can find the subject in the certificate's content.

    ```bash
    openssl req -new -sha256 -subj "/CN=yourcn" -key cert-key.pem -out cert.csr
    ```

3. Create a file with all the alternative names

    ```bash
    echo "subjectAltName=DNS:your.dns,IP:172.16.1.4,IP:fe80::b3df:fd02:8d8f:49c5" >> extfilecnf
    echo "extendedKeyUsage=serverAuth" >> extFile.cnf
    ```

4. Create the certificate

    > You should specify the desire **-days** for expiration.

    > You’ll need to supply the CA’s passphrase to complete the process.

    ```bash
    openssl x509 -req -sha256 -days 365 -in cert.csr -CA ca.pem -CAkey ca-key.pem -out cert.pem -extfile extfile.cnf -CAcreateserial
    ```

5. Verify certificates
    ```bash
    openssl verify -CAfile ca.pem -verbose cert.pem
    ```

### Convert Certs

| COMMAND                                                | CONVERSION         |
| ------------------------------------------------------ | ------------------ |
| `openssl x509 -outform der -in cert.pem -out cert.der` | **PEM** to **DER** |
| `openssl x509 -inform der -in cert.der -out cert.pem`  | **DER** to **PEM** |
| `openssl pkcs12 -in cert.pfx -out cert.pem -nodes`     | **PFX** to **PEM** |

### Install the CA Cert as a trusted root CA

#### Windows

-   Using PowerShell you can use this command (with admin permission):

    ```powershell
    Import-Certificate -FilePath "path_to_ca.pem" -CertStoreLocation Cert:\LocalMachine\Root
    ```

    -   In case you want to trust certificates only for the logged in user set
        `-CertStoreLocation` to `Cert:\CurrentUser\Root` .

-   Using Command Prompt you can use this command:

    ```bash
    certutil.exe -addstore root C:\ca.pem
    ```
