# OpenSSL PKI — Root CA, Intermediate CA and Certificate Chains
Student Name: Mohammed Omer Ahmed

Student ID: 12303667

Project: OpenSSL-CA-12303667

# 1. Overview
This activity involved building a two-tier Public Key Infrastructure (PKI) using OpenSSL. A Root CA and Intermediate CA were created, and the Intermediate CA was used to sign a web server certificate. The certificate chain was then verified from a client, and HTTPS traffic was tested and captured.

# 2. Network Setup
Three Linux hosts were connected through an Ethernet switch.

| Host | IP Address | Purpose |
|------|------------|---------|
| CA | 10.10.1.10/24 | Root and Intermediate CA |
| Server | 10.10.1.20/24 | HTTPS web server |
| Client | 10.10.1.30/24 | Certificate verification |

 # Evidence
 
 <img width="1919" height="1079" alt="19 week 2" src="https://github.com/user-attachments/assets/a55865e6-3cc8-48c4-ac7a-8ada14b6c737" />

<img width="1919" height="1079" alt="2 week 2" src="https://github.com/user-attachments/assets/aff18453-bea0-4859-b5b0-b113b58f5d4b" />



 # 3. Root and Intermediate CA

On the CA host, the Root CA directory structure was created and a 4096-bit RSA private key and self-signed Root CA certificate were generated.

An Intermediate CA was then created with its own private key and CSR. The Root CA signed the Intermediate CA certificate, creating the two-tier trust hierarchy:

Root CA

   ↓
   
Intermediate CA

   ↓
   
Server Certificate

Using an Intermediate CA means the Root CA does not need to directly sign everyday server certificates.

# 4. Server Certificate
- On the Server, a 2048-bit RSA private key and CSR were generated for:

www.12303667.lab

- The certificate included the required Subject Alternative Name (SAN):

subjectAltName=DNS:www.12303667.lab

The CSR was signed by the Intermediate CA to produce the server certificate.

A CA chain containing the Intermediate and Root certificates was also created.

<img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/0fc1817c-7577-49ed-a000-99a756a203ed" />


# 5. Nginx HTTPS Configuration

- The server certificate and CA chain were combined into a full certificate chain:

cat /tmp/server.crt /tmp/ca-chain.crt > /etc/ssl/certs/server-fullchain.crt

- Nginx was configured to use:

ssl_certificate /etc/ssl/certs/server-fullchain.crt;

ssl_certificate_key /etc/ssl/private/server.key;

The configuration was tested and Nginx was restarted successfully.

# Evidence
<img width="1600" height="901" alt="image" src="https://github.com/user-attachments/assets/3f4a78f5-d524-4f39-a579-d634e79fc391" />


# 6. Certificate Verification

- On the Client, the domain was mapped to the Server IP:

10.10.1.20 www.12303667.lab

- The certificate chain was verified using:

openssl verify -CAfile /tmp/root-ca.crt -untrusted /tmp/intermediate.crt /tmp/server.crt

- Result:

/tmp/server.crt: OK

This confirmed that the Server certificate could be successfully traced through the Intermediate CA to the trusted Root CA.

# Evidence
<img width="1600" height="892" alt="image" src="https://github.com/user-attachments/assets/e7207f95-45a2-42d5-a43a-c4e5b660ad0f" />


# 7. HTTPS Testing

- The HTTPS connection was tested using:

curl --cacert /tmp/root-ca.crt https://www.12303667.lab/

The request completed successfully, confirming that the Client trusted the certificate chain and could securely communicate with the Server.

- The live TLS certificate chain was also inspected using:

openssl s_client -connect www.12303667.lab:443 -CAfile /tmp/root-ca.crt

# Evidence
<img width="1919" height="1079" alt="8 week 2" src="https://github.com/user-attachments/assets/6d0b8499-cd3f-4205-8362-f5d9edf1e770" />


# 8. Packet Capture

A packet capture was taken between the switch and Server while the Client made another HTTPS request.

# Evidence
<img width="1919" height="1079" alt="1 week 2" src="https://github.com/user-attachments/assets/d76acfe3-e622-4f5a-b875-76d61c7a458d" />


The capture showed the TLS handshake and certificate exchange. However, the HTTP request content was encrypted and therefore could not be read directly from the capture.

# Analysis Questions
- What is the purpose of an Intermediate CA?

An Intermediate CA signs server certificates on behalf of the Root CA. This is safer because the Root CA private key can be kept protected and used less frequently. If an Intermediate CA is compromised, it can be revoked without replacing the Root CA.

- What does openssl verify check?

openssl verify checks whether the server certificate can form a valid chain through the Intermediate CA to the trusted Root CA. If the required Intermediate certificate is missing, OpenSSL cannot complete the certificate path and verification will fail.

- Why is HTTP content not visible in the packet capture?

During the TLS handshake, certificates are exchanged so the Client can authenticate the Server and establish encryption. Once the secure TLS session is established, application data such as HTTP requests and responses is encrypted. Therefore, the packet capture can observe TLS traffic but cannot directly read the HTTPS content.

- Self-Signed vs CA-Signed Certificates:

A self-signed certificate is signed using its own private key and is not automatically trusted by clients. It is generally suitable for testing, laboratories and some controlled internal environments.

A CA-signed certificate is signed by a trusted CA and provides a chain of trust. It is more appropriate for production systems and services where clients need to verify the identity of the server.

# Conclusion

This activity demonstrated how PKI provides trust using a Root CA, Intermediate CA and server certificate. The successful OpenSSL verification and HTTPS connection confirmed that the complete certificate chain was working correctly. The packet capture also demonstrated how TLS protects application data during network communication.
