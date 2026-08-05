# 🚀 Lab 02 – Host a Web Server on Amazon EC2

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Apache](https://img.shields.io/badge/Web%20Server-Apache-red)
![HTTPS](https://img.shields.io/badge/HTTPS-Enabled-success)
![Reverse Proxy](https://img.shields.io/badge/Reverse%20Proxy-Apache-blue)
![Python](https://img.shields.io/badge/Python-Flask-yellow)

---

# Overview

This lab extends the EC2 instance created in **Lab 01 – Host an Application on Amazon EC2**.

Instead of launching another EC2 instance, the existing **App-Ec2** instance was enhanced by enabling HTTPS, configuring Apache SSL, deploying a Python Flask backend application, and configuring Apache Reverse Proxy.

The static website deployed during Lab 01 continues to be served by Apache, while a separate Python backend application is accessible through the **/backend** endpoint.

---

# Architecture

```text
                         Internet
                  HTTP (80) / HTTPS (443)
                           │
                           ▼
                  Apache HTTP Server
                 Amazon Linux 2023 EC2
                           │
          ┌────────────────┴────────────────┐
          │                                 │
          ▼                                 ▼
     Static Website                    Reverse Proxy
    (/var/www/html)                    (/backend)
          │                                 │
          ▼                                 ▼
  EC2 User Data Application          Python Flask
                                     127.0.0.1:8000
```

---

# Lab Objectives

- Reuse the EC2 instance created in Lab 01
- Configure HTTPS using Apache SSL
- Update the Security Group to allow HTTPS traffic
- Create a Python Flask backend application
- Configure Apache Reverse Proxy
- Validate secure access over HTTPS

---

# Existing Environment

The following infrastructure was created during **Lab 01** and reused in this lab.

| Resource | Configuration |
|----------|---------------|
| EC2 Instance | App-Ec2 |
| Operating System | Amazon Linux 2023 |
| Web Server | Apache HTTP Server |
| Static Website | `/var/www/html` |
| Deployment Method | EC2 User Data |
| Security Group | app-sg |

---

# Step 1 – Verify Existing Apache Installation

Verify Apache is running.

```bash
sudo systemctl status httpd
```


```text
Active: active (running)
```

Verify Apache listening ports.

```bash
sudo ss -tulpn | grep httpd
```


```text
*:80
```

---

# Step 2 – Install Apache SSL Module

Install SSL support.

```bash
sudo dnf install mod_ssl -y
```

Verify installation.

```bash
ls /etc/httpd/conf.d
```


```text
ssl.conf
```

Verify SSL certificate configuration.

```bash
grep -E "SSLCertificateFile|SSLCertificateKeyFile" /etc/httpd/conf.d/ssl.conf
```


```text
SSLCertificateFile /etc/pki/tls/certs/localhost.crt
SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
```

Restart Apache.

```bash
sudo systemctl restart httpd
```

Verify HTTPS listener.

```bash
sudo ss -tulpn | grep httpd
```


```text
*:80
*:443
```

---

# Step 3 – Configure Security Group

Update the EC2 Security Group by adding an inbound rule.

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| HTTPS | TCP | 443 | 0.0.0.0/0 |

After updating the Security Group, verify HTTPS.

```
https://<Public-IP>
```

The browser displays the existing Lab 01 application over HTTPS.

> **Note:** A browser warning is expected because a self-signed SSL certificate is used.

---

# Step 4 – Create the Python Backend Application

Create a separate working directory.

```bash
mkdir ~/backend-app
cd ~/backend-app
```

Create the application.

```bash
nano app.py
```

Paste the following code.

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return """
    <html>
    <head>
        <title>Python Backend</title>
    </head>
    <body style="font-family:Arial;text-align:center;margin-top:100px;">
        <h1>Reverse Proxy Working Successfully!</h1>
        <h2>Python Flask Backend</h2>
        <p>This application is running on port 8000.</p>
    </body>
    </html>
    """

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=8000)
```
---

# Step 5 – Create the Python Backend Application

After successfully configuring HTTPS, the next step is to create a backend application that Apache will forward requests to using Reverse Proxy.

Instead of modifying the existing static website deployed during **Lab 01**, a separate Python backend application was created. This approach keeps the original application intact while allowing Apache to proxy requests to another application.

## Create a Backend Directory

Navigate to the home directory.

```bash
cd ~
```

Create a new directory for the backend application.

```bash
mkdir backend-app
```

Move into the directory.

```bash
cd backend-app
```

Verify the current working directory.

```bash
pwd
```

Output

```text
/home/ec2-user/backend-app
```

---

## Create the Flask Application

Create a new Python file.

```bash
nano app.py
```

Paste the following application code.

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return """
    <html>
    <head>
        <title>Python Backend</title>
    </head>
    <body style="font-family:Arial;text-align:center;margin-top:100px;">
        <h1>Reverse Proxy Working Successfully!</h1>
        <h2>Python Flask Backend</h2>
        <p>This application is running on port 8000.</p>
    </body>
    </html>
    """

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=8000)
```

Save the file.

For Nano

- Press **Ctrl + O**
- Press **Enter**
- Press **Ctrl + X**

---

## Verify the Application File

List the files.

```bash
ls -l
```

Output

```text
-rw-r--r--. 1 ec2-user ec2-user app.py
```

Display the file content.

```bash
cat app.py
```

Verify the application code has been saved correctly before continuing.

> **Note**
>
> The backend application is created under **/home/ec2-user/backend-app** instead of **/var/www/html**.
>
> The existing static website created during Lab 01 remains unchanged while Apache forwards requests to this backend application using Reverse Proxy.

---

# Step 6 – Install Flask and Run the Backend Application

The Flask framework is required to run the Python backend application.

## Verify Python Installation

Check the installed Python version.

```bash
python3 --version
```

Output

```text
Python 3.x.x
```

---

## Verify pip Installation

Check the pip version.

```bash
python3 -m pip --version
```

If pip is not installed, install it.

```bash
sudo dnf install python3-pip -y
```

Verify again.

```bash
python3 -m pip --version
```

---

## Install Flask

Install Flask using pip.

```bash
python3 -m pip install flask
```

Expected Output

```text
Successfully installed flask
```

Verify the installation.

```bash
python3 -m pip show flask
```

Output

```text
Name: Flask
Version: <version>
Location: ...
```

---

## Run the Flask Application

Start the backend application.

```bash
python3 app.py
```

Output

```text
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://127.0.0.1:8000
Press CTRL+C to quit
```

> **Important**
>
> Leave this terminal running.
>
> Do **not** press **Ctrl + C**, as it will stop the Flask application.

---

## Open a New SSH Session

Open a second terminal and connect to the same EC2 instance.

```powershell
ssh -i "App-key.pem" ec2-user@<Public-IP>
```

The first SSH session continues running the Flask application, while the second session is used to configure Apache.

---

## Verify the Backend Application

Verify the backend is accessible locally.

```bash
curl http://127.0.0.1:8000
```

Output

```html
<html>
<head>
<title>Python Backend</title>
</head>

<body>

<h1>Reverse Proxy Working Successfully!</h1>

<h2>Python Flask Backend</h2>

<p>This application is running on port 8000.</p>

</body>
</html>
```
# Step 7 – Verify Apache Proxy Modules

Before configuring Apache as a Reverse Proxy, verify that the required proxy modules are available.

Run the following command.

```bash
sudo httpd -M | grep proxy
```

Output

```text
proxy_module (shared)
proxy_http_module (shared)
proxy_ajp_module (shared)
proxy_balancer_module (shared)
proxy_connect_module (shared)
proxy_express_module (shared)
proxy_fcgi_module (shared)
proxy_fdpass_module (shared)
proxy_ftp_module (shared)
proxy_hcheck_module (shared)
proxy_scgi_module (shared)
proxy_uwsgi_module (shared)
proxy_wstunnel_module (shared)
proxy_http2_module (shared)
```

Verify the following modules are available.

- proxy_module
- proxy_http_module

These modules are required to configure Apache Reverse Proxy.

---

# Step 8 – Configure Apache Reverse Proxy

Create a new Apache configuration file.

```bash
sudo nano /etc/httpd/conf.d/reverse-proxy.conf
```

Add the following configuration.

```apache
ProxyRequests Off

ProxyPass /backend http://127.0.0.1:8000/
ProxyPassReverse /backend http://127.0.0.1:8000/
```

Save the file.

For Nano

- Ctrl + O
- Enter
- Ctrl + X

---

# Step 9 – Validate Apache Configuration

Before restarting Apache, validate the configuration.

```bash
sudo httpd -t
```

Output

```text
Syntax OK
```

If any configuration errors are displayed, correct them before continuing.

---

# Step 10 – Restart Apache

Restart Apache to apply the Reverse Proxy configuration.

```bash
sudo systemctl restart httpd
```

Verify Apache is running.

```bash
sudo systemctl status httpd
```

Output

```text
Active: active (running)
```

Verify Apache is listening on both HTTP and HTTPS.

```bash
sudo ss -tulpn | grep httpd
```

Output

```text
*:80
*:443
```

---

# Step 11 – Validate Reverse Proxy

Verify the backend application directly.

```bash
curl http://127.0.0.1:8000
```

Output

```html
<h1>Reverse Proxy Working Successfully!</h1>
```

Now verify Apache Reverse Proxy.

```bash
curl http://localhost/backend
```

Output

```html
<h1>Reverse Proxy Working Successfully!</h1>
```

Apache is now successfully forwarding requests to the backend Flask application.

---

# Step 12 – Browser Validation

Open the existing application.

```
http://<Public-IP>
```

or

```
https://<Public-IP>
```

Result

```
Application Hosted Successfully!
```

This confirms the original Lab 01 application is still available.

Now open the backend application.

```
http://<Public-IP>/backend
```

or

```
https://<Public-IP>/backend
```

Result

```
Reverse Proxy Working Successfully!

Python Flask Backend

This application is running on port 8000.
```

This confirms Apache is acting as a Reverse Proxy and forwarding requests to the backend application.

---

# Deployment Validation

| Validation | Status |
|------------|--------|
| Apache Running | ✅ |
| HTTPS Configured | ✅ |
| Port 443 Enabled | ✅ |
| Python Installed | ✅ |
| Flask Installed | ✅ |
| Backend Application Running | ✅ |
| Apache Proxy Modules Verified | ✅ |
| Reverse Proxy Configured | ✅ |
| Reverse Proxy Validated | ✅ |
| Browser Validation Completed | ✅ |

---

# Learning Outcomes


- Apache HTTPS Configuration
- Apache SSL Module (mod_ssl)
- Self-Signed SSL Certificates
- Security Group Configuration for HTTPS
- Python Flask Application Deployment
- Apache Reverse Proxy
- ProxyPass and ProxyPassReverse
- Backend Application Validation
- Reverse Proxy Architecture
- Hosting Multiple Applications on a Single EC2 Instance

---


