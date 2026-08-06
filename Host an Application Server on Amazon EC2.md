# 🚀 Lab 03 – Host an Application Server on Amazon EC2

![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Application Server](https://img.shields.io/badge/Application-Server-blue)
![Python](https://img.shields.io/badge/Python-3.9-blue)
![Flask](https://img.shields.io/badge/Framework-Flask-green)
![Gunicorn](https://img.shields.io/badge/Gunicorn-Application%20Server-red)
![systemd](https://img.shields.io/badge/Linux-systemd-success)
![Virtual Environment](https://img.shields.io/badge/Python-Virtual%20Environment-purple)

---

# 🎯 Lab Objectives

The objectives of this lab are:

- Reuse the existing Amazon EC2 instance.
- Install the Python application runtime.
- Configure a Python Virtual Environment.
- Deploy a Flask application using Gunicorn.
- Configure the application as a Linux **systemd** service.
- Validate that the application server is running successfully.

---

# 🌐 Existing Environment

The existing EC2 instance from the previous labs is reused for this implementation.

| Previous Lab | Implementation |
|--------------|----------------|
| Lab 01 | Static Website hosted using Apache HTTP Server |
| Lab 02 | Flask Backend Application configured using Apache Reverse Proxy |
| Lab 03 | Python Application Server using Flask, Gunicorn and systemd |

Instead of creating another EC2 instance, a separate directory named **application-server** is created under the existing user's home directory. This approach keeps all three labs isolated while using the same infrastructure.

```

# 🏗️ Architecture

```text
                           Internet
                               │
                               ▼
                    Amazon EC2 (Amazon Linux 2023)
                               │
         ┌─────────────────────┼──────────────────────┐
         │                     │                      │
         ▼                     ▼                      ▼

   /var/www/html        /home/ec2-user/        /home/ec2-user/
      (Lab 01)            backend-app        application-server
                              (Lab 02)             (Lab 03)
         │                     │                      │
         ▼                     ▼                      ▼

  Apache HTTP Server     Flask Backend        Python Virtual Environment
  Static Website         (Development)                │
                                                      ▼
                                                 Gunicorn Server
                                                      │
                                                      ▼
                                               Flask Application
                                                      │
                                                      ▼
                                     systemd Linux Service

```

# Step 1 – Create the Application Server Directory

The existing EC2 instance from **Lab 01** and **Lab 02** is reused in this lab.

Instead of deploying another EC2 instance, a separate directory is created to host the Python application server. This keeps the existing static website and backend application unchanged while deploying a dedicated application server.

Navigate to the home directory.

```bash
cd ~
```

Create the application server directory.

```bash
mkdir application-server
```

Move into the directory.

```bash
cd application-server
```

Verify the current working directory.

```bash
pwd
```

Output

```text
/home/ec2-user/application-server
```

---

# Step 2 – Configure Python Virtual Environment

The application server requires an isolated Python environment to manage application dependencies independently from the operating system.

Verify the installed Python version.

```bash
python3 --version
```

Output

```text
Python 3.9.25
```

Create a Python Virtual Environment.

```bash
python3 -m venv venv
```

Verify the environment.

```bash
ls -l
```

Output

```text
venv/
```

Activate the Virtual Environment.

```bash
source venv/bin/activate
```

Output

```text
(venv)
```

> **Note**
>
> The Virtual Environment isolates the application dependencies from the system Python installation. This allows different applications on the same EC2 instance to use different package versions without conflicts.

---

# Step 3 – Install the Application Runtime

The application server requires Flask to run the application and Gunicorn to serve the application in a production-style environment.

Install Flask and Gunicorn.

```bash
pip install flask gunicorn
```

Verify Flask.

```bash
pip show flask
```

Verify Gunicorn.

```bash
pip show gunicorn
```

Generate the dependency file.

```bash
pip freeze > requirements.txt
```

Verify.

```bash
cat requirements.txt
```

---

# Step 4 – Create the Flask Application

Create the application file.

```bash
nano app.py
```

Paste the following application.

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return """
    <html>
    <head>
        <title>Application Server</title>
    </head>
    <body style="font-family:Arial;text-align:center;margin-top:100px;">
        <h1>Application Server Running Successfully!</h1>
        <h2>Python Flask + Gunicorn</h2>
        <p>Application Runtime Installed Successfully.</p>
        <p>Running inside Python Virtual Environment.</p>
    </body>
    </html>
    """

if __name__ == "__main__":
    app.run()
```

Save the file.

For Nano

- Ctrl + O
- Enter
- Ctrl + X

Verify the file.

```bash
cat app.py
```

---

# Step 5 – Test the Application Using Gunicorn

Before configuring the application as a Linux service, verify that the Flask application runs successfully using Gunicorn.

Start the application.

```bash
gunicorn --bind 127.0.0.1:8001 app:app
```

Output

```text
Starting gunicorn...
Listening at: http://127.0.0.1:8001
```

Open a new SSH session and verify the application.

```bash
curl http://127.0.0.1:8001
```

Output

```html
<h1>Application Server Running Successfully!</h1>
```

After validation, stop the manually started Gunicorn process.

```text
Ctrl + C
```

> **Note**
>
> Gunicorn is stopped because the application will be managed by a Linux systemd service in the next step.

```
# Step 6 – Configure the Application as a Linux Service

The Flask application has been successfully validated using Gunicorn.

However, the application is currently running manually from the terminal. If the SSH session is closed or the EC2 instance is restarted, the application will stop.

To ensure the application starts automatically and runs as a background service, configure it as a Linux **systemd** service.

Create a new service file.

```bash
sudo nano /etc/systemd/system/application-server.service
```

Add the following configuration.

```ini
[Unit]
Description=Python Flask Application Server
After=network.target

[Service]
User=ec2-user
Group=ec2-user

WorkingDirectory=/home/ec2-user/application-server

Environment="PATH=/home/ec2-user/application-server/venv/bin"

ExecStart=/home/ec2-user/application-server/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:8001 app:app

Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Save the file.

For Nano

- Ctrl + O
- Enter
- Ctrl + X

---

# Step 7 – Reload systemd Configuration

Reload the systemd daemon to detect the newly created service.

```bash
sudo systemctl daemon-reload
```

Enable the application service.

```bash
sudo systemctl enable application-server
```

Output

```text
Created symlink /etc/systemd/system/multi-user.target.wants/application-server.service → /etc/systemd/system/application-server.service
```

Start the service.

```bash
sudo systemctl start application-server
```

---

# Step 8 – Validate the Application Service

Verify the application service.

```bash
sudo systemctl status application-server
```

Output

```text
Active: active (running)
```

The output also confirms that Gunicorn is running as a Linux service.

```text
Listening at: http://127.0.0.1:8001
Using worker: sync
Booting worker with pid ...
```

---

# Step 9 – Verify the Application

Verify the application locally.

```bash
curl http://127.0.0.1:8001
```

Output

```html
<html>
<head>
<title>Application Server</title>
</head>

<body>

<h1>Application Server Running Successfully!</h1>

<h2>Python Flask + Gunicorn</h2>

<p>Application Runtime Installed Successfully.</p>

<p>Running inside Python Virtual Environment.</p>

</body>
</html>
```

Verify Gunicorn is listening on port **8001**.

```bash
sudo ss -tulpn | grep 8001
```

Output

```text
tcp LISTEN 0 2048 127.0.0.1:8001
users:(("gunicorn",pid=xxxx))
```

This confirms the application server is successfully running as a Linux service.

---

# Step 10 – Deployment Validation

| Validation | Status |
|------------|--------|
| Existing EC2 Instance Reused | ✅ |
| Python Runtime Installed | ✅ |
| Python Virtual Environment Configured | ✅ |
| Flask Installed | ✅ |
| Gunicorn Installed | ✅ |
| requirements.txt Generated | ✅ |
| Flask Application Created | ✅ |
| Gunicorn Validated | ✅ |
| systemd Service Created | ✅ |
| Service Enabled | ✅ |
| Service Running | ✅ |
| Application Validated | ✅ |

---

# Learning Outcomes

- Python Virtual Environment
- Python Package Management
- Flask Application Deployment
- Gunicorn Application Server
- Linux systemd Service
- Application Runtime Configuration
- Production-style Python Application Hosting
- Service Management using systemctl

---

# Troubleshooting

## Gunicorn Port Already in Use

While configuring the systemd service, the following error was encountered.

```text
Connection in use: ('127.0.0.1', 8001)
```

### Cause

A manually started Gunicorn process was already running on port **8001**.

### Resolution

Stop the manually started Gunicorn process.

```bash
pkill gunicorn
```

Restart the application service.

```bash
sudo systemctl restart application-server
```

Verify again.

```bash
sudo systemctl status application-server
```

The service should now display.

```text
Active: active (running)
```

---

# Conclusion

In this lab, the existing Amazon EC2 instance was reused to deploy a Python application server.

A dedicated application directory was created, followed by the configuration of a Python Virtual Environment. Flask and Gunicorn were installed to provide the application runtime, and the Flask application was deployed using Gunicorn.

Finally, the application was configured as a Linux **systemd** service, ensuring it starts automatically during system boot and continues running in the background.

This completes the **Host an Application Server on Amazon EC2** task.

---

