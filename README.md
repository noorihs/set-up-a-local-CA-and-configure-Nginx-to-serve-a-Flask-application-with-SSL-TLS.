# **🔐 Flask App with SSL Certificate Signed by Local CA**

## **📖 Description**

This project shows how to create a secure **Flask** web app using **HTTPS** with a certificate signed by a **local Certificate Authority (CA)**. The CA is created manually using **OpenSSL**, and the Flask app is served through **Nginx** as a reverse proxy.

---

## **✅ Requirements**

- Linux (tested on Kali)
- OpenSSL
- Nginx
- Python3 + Flask
- `systemd` (to manage services)

---

## **🛠️ Step-by-Step Guide**

### **🔸 1. Create the Local Certificate Authority (CA)**

```bash
cd ~/Desktop/ssl-srt
mkdir CA_folder
cd CA_folder
openssl req -x509 -newkey rsa:4096 -sha256 -days 365 \
-keyout local-ca.key -out local-ca.crt
```

When prompted, fill in with something like:

```
Country: DZ
State: algiers
City: kouba
Organization: CAihsene
Unit: ihsene
Common Name: CAihsene.com
```

---

### **🔸 2. Generate the Flask App Certificate**

```bash
cd ..
mkdir flask_crt
cd flask_crt
openssl genrsa -out server.key 4098
openssl req -new -key server.key -out server.csr \
-subj "/C=DZ/ST=State/L=City/O=LocalOrg/OU=FlaskApp/CN=flask-app.local"
openssl x509 -req -in server.csr \
-CA ../CA_folder/local-ca.crt -CAkey ../CA_folder/local-ca.key \
-CAcreateserial -days 365 -sha256 -extfile localhost.ext -out flask-app.crt
cat flask-app.crt ../CA_folder/local-ca.crt > flask-app-bundle.crt
```

---

### **🔸 3. Add Domain to /etc/hosts**

```bash
sudo nano /etc/hosts
```

Add this line:

```
127.0.0.1    flask-app.local
```

---

### **🔸 4. Configure Nginx**

Check if Nginx is running:

```bash
sudo systemctl status nginx
```

Start it if it's not:

```bash
sudo systemctl start nginx
```

Check if Apache is running (and stop it if needed):

```bash
sudo systemctl status apache2.service
```

Now create a new Nginx site config:

```bash
sudo nano /etc/nginx/sites-available/flask-app
```

Paste this inside:

```nginx
server {
    listen 443 ssl;
    server_name flask-app.local;

    ssl_certificate /home/ihsene/Desktop/ssl-srt/flask_crt/flask-app-bundle.crt;
    ssl_certificate_key /home/ihsene/Desktop/ssl-srt/flask_crt/server.key;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Then enable the site and reload:

```bash
sudo ln -s /etc/nginx/sites-available/flask-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### **🔸 5. Create and Run the Flask App**

Create the app file:

```bash
nano app.py
```

Paste this code:

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Flask App with SSL!"

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=5000)
```

Run it:

```bash
python3 app.py
```

---

## **🌐 Open the App**

Visit:

```bash
https://flask-app.local
```

You may get a browser warning since it's signed by a local CA.

---

## **🧰 Troubleshooting**

**❌ Nginx certificate errors?**

- Double check the paths to your `.crt` and `.key` files.

**❌ Flask app not responding?**

- Make sure it's running on port `5000`.
- Nginx is only a reverse proxy; it doesn’t launch the Flask app.

---

## **✅ What I Learned**

- Creating a local Certificate Authority (CA)
- Signing SSL certificates manually with OpenSSL
- Using HTTPS with Flask
- Configuring Nginx as a reverse proxy

This was a hands-on way to understand how HTTPS works under the hood!

