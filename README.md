# Flask Application with SSL/TLS using a Local Certificate Authority (CA)

This project demonstrates how to set up a local Certificate Authority (CA), generate SSL/TLS certificates for a Flask application, and configure Nginx as a reverse proxy to serve the application securely with HTTPS.

### Steps Taken:

## 1. **Creating a Local Certificate Authority (CA)**

First, we created a local Certificate Authority (CA) to act as the root authority for signing our SSL certificates.

### Generate a CA certificate and key:

Run the following command to create the certificate (`local-ca.crt`) and the private key (`local-ca.key`) for our local CA:

```bash
openssl req -x509 -newkey rsa:4096 -sha256 -days 365 -keyout local-ca.key -out local-ca.crt
