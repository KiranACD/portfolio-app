# Portfolio App

Personal portfolio website for Kiran Ramachandran — software developer and systematic trader.

Built with Python/Flask serving a single-page HTML/CSS/JS frontend.

## Requirements

- Python 3.8+
- pip

## Local Development

```bash
# Clone the repository
git clone https://github.com/KiranACD/portfolio-app.git
cd portfolio-app

# Create and activate a virtual environment
python3 -m venv venv
source venv/bin/activate        # macOS/Linux
# venv\Scripts\activate         # Windows

# Install dependencies
pip install -r requirements.txt

# Run the development server
python app.py
```

The app will be available at `http://localhost:5000`.

## Steps to Deploy

### 1. Provision a server

Spin up a Linux VM (e.g. AWS EC2 `t3.micro`, Ubuntu 22.04 LTS). Open inbound ports **22** (SSH) and **80** / **443** (HTTP/HTTPS) in the security group.

### 2. Install dependencies on the server

```bash
sudo apt update && sudo apt install -y python3 python3-pip python3-venv nginx
```

### 3. Clone the repository

```bash
cd /home/ubuntu
git clone https://github.com/KiranACD/portfolio-app.git
cd portfolio-app
```

### 4. Set up the Python environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install gunicorn
```

### 5. Run with Gunicorn

```bash
gunicorn --workers 2 --bind 127.0.0.1:8000 app:app
```

To keep the process running after logout, use a systemd service:

```ini
# /etc/systemd/system/portfolio.service
[Unit]
Description=Portfolio Flask App
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/portfolio-app
ExecStart=/home/ubuntu/portfolio-app/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable portfolio
sudo systemctl start portfolio
```

### 6. Configure Nginx as a reverse proxy

```nginx
# /etc/nginx/sites-available/portfolio
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /static/ {
        alias /home/ubuntu/portfolio-app/static/;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/portfolio /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 7. (Optional) Enable HTTPS with Let's Encrypt

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

Certbot will automatically update the Nginx config and schedule certificate renewal.

## Project Structure

```
portfolio-app/
├── app.py              # Flask application entry point
├── requirements.txt    # Python dependencies
├── templates/
│   └── index.html      # Single-page portfolio template
├── static/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── main.js
└── docs/
    └── kiran-resume-2.pdf
```
