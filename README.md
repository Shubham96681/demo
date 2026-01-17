# Flask CI/CD Demo Project (AWS EC2)

This is a **small, production-style demo project** to teach students how **Flask + GitHub + CI/CD + EC2** work end-to-end.

---

## 1. Project Overview

**What students will learn**

* Build a basic Flask app
* Run Flask using Gunicorn
* Deploy on AWS EC2
* Automate deployment using GitHub Actions (CI/CD)

**Tech Stack**

* Python 3
* Flask
* Gunicorn
* Nginx
* AWS EC2 (Ubuntu)
* GitHub Actions

---

## 2. Project Structure

```
flask-cicd-demo/
│── app.py
│── requirements.txt
│── wsgi.py
│── README.md
│── .github/
│   └── workflows/
│       └── deploy.yml
```

---

## 3. Flask Application Code

### app.py

The main Flask application with two routes:
- `/` - Home route
- `/health` - Health check endpoint

### wsgi.py

WSGI entry point for Gunicorn server.

### requirements.txt

Python dependencies including Flask and Gunicorn.

---

## 4. Create EC2 Server (Manual – Demo Friendly)

1. Launch **EC2 Ubuntu 22.04**
2. Instance type: `t2.micro`
3. Open Security Group ports:
   * 22 (SSH)
   * 80 (HTTP)

---

## 5. EC2 Server Setup

### Connect to EC2

```bash
ssh -i key.pem ubuntu@<EC2_PUBLIC_IP>
```

### Install dependencies

```bash
sudo apt update -y
sudo apt install python3-pip python3-venv nginx git -y
```

---

## 6. Clone Project on EC2

```bash
cd /var/www
sudo git clone https://github.com/<your-username>/flask-cicd-demo.git
sudo chown -R ubuntu:ubuntu flask-cicd-demo
cd flask-cicd-demo
```

---

## 7. Python Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

## 8. Run Flask with Gunicorn (Test)

```bash
gunicorn --bind 0.0.0.0:5000 wsgi:app
```

Visit:

```
http://EC2_PUBLIC_IP:5000
```

---

## 9. Configure Nginx

### Create config

```bash
sudo nano /etc/nginx/sites-available/flask
```

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/flask /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

---

## 10. GitHub Actions CI/CD Pipeline

### .github/workflows/deploy.yml

The GitHub Actions workflow automatically deploys to EC2 when code is pushed to the `main` branch.

**Features:**
- Triggers on push to `main` branch
- SSH into EC2 server
- Pull latest code from GitHub
- Update dependencies
- Restart Gunicorn server

---

## 11. GitHub Secrets (Very Important)

Add these in **GitHub → Settings → Secrets → Actions**

| Name        | Value               |
| ----------- | ------------------- |
| EC2_HOST    | EC2 Public IP       |
| EC2_SSH_KEY | Private key content |

**How to add secrets:**
1. Go to your GitHub repository
2. Navigate to Settings → Secrets and variables → Actions
3. Click "New repository secret"
4. Add each secret with the exact names above

---

## 12. CI/CD Flow (Explain to Students)

1. Developer pushes code to GitHub
2. GitHub Actions pipeline triggers
3. SSH into EC2
4. Pulls latest code
5. Restarts Gunicorn
6. App updated automatically

---

## 13. Demo Test

Change text in `app.py`, push to GitHub:

```bash
git add .
git commit -m "Update message"
git push origin main
```

Refresh browser → **New change visible without manual deploy**

---

## 14. Interview Talking Points

* **Why Gunicorn over Flask dev server?** - Production-ready WSGI server with better performance, process management, and security.
* **Why CI/CD is required?** - Automates deployment, reduces errors, enables rapid iteration, and ensures consistency.
* **Role of Nginx** - Reverse proxy that handles HTTP requests, SSL termination, load balancing, and static file serving.
* **GitHub Secrets security** - Protects sensitive credentials by storing them encrypted and only accessible to GitHub Actions workflows.

---

## 15. Optional Enhancements

* Add Docker
* Add systemd service
* Add HTTPS (Let's Encrypt)
* Add health check monitoring

---

### ✅ This project is perfect for classroom demo + interviews

If you want:
* **Docker version**
* **Terraform-based EC2**
* **Jenkins CI/CD**

Tell me 👍

---

## Quick Start

1. Clone this repository
2. Create an EC2 instance (Ubuntu 22.04)
3. Set up the server following steps 5-9
4. Configure GitHub Secrets (step 11)
5. Push code to `main` branch to trigger deployment

---

## Troubleshooting

**Issue:** Cannot connect to EC2 via SSH
- Check security group allows port 22
- Verify SSH key permissions: `chmod 400 key.pem`

**Issue:** Gunicorn not starting
- Check if port 5000 is already in use: `sudo lsof -i :5000`
- Verify virtual environment is activated
- Check Gunicorn logs

**Issue:** Nginx not proxying correctly
- Test Nginx config: `sudo nginx -t`
- Check Nginx error logs: `sudo tail -f /var/log/nginx/error.log`
- Verify Gunicorn is running on 127.0.0.1:5000

---

## License

This project is for educational purposes.

