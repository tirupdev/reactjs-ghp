
# 🚀 React App Deployment on EC2 with GitHub Actions (Self-Hosted)

This guide explains how to create, build, and deploy a React.js app using a self-hosted GitHub Actions runner on an EC2 instance with NGINX.

---

## 🧰 Prerequisites

- ✅ AWS EC2 instance (Ubuntu)
- ✅ Node.js and npm installed
- ✅ NGINX installed and running
- ✅ Self-hosted GitHub Actions runner installed
- ✅ Port 80 open in EC2 security group
- ✅ Your React project is pushed to GitHub

---

## 📁 React Project Structure (After Build)

```
my-react-app/
├── node_modules/
├── public/
├── src/
├── build/                    ← Generated on `npm run build`
│   ├── index.html
│   ├── asset-manifest.json
│   └── static/
├── package.json
├── .github/
│   └── workflows/
│       └── deploy.yml        ← GitHub Actions workflow
```

---

## 🔧 EC2 Setup

```bash
# On EC2 terminal
sudo apt update
sudo apt install -y nginx nodejs npm

# Optional: link node to nodejs if missing
sudo ln -s /usr/bin/node /usr/bin/nodejs
```

---

## 🔥 React Build and Deployment Steps (CI/CD)

### 1. Update `package.json`

```json
"homepage": "http://<your-ec2-public-ip>"
```

---

### 2. GitHub Actions Workflow: `.github/workflows/deploy.yml`

```yaml
name: Build and Deploy React App

on:
  push:
    branches: [ main ]

jobs:
  build-deploy:
    runs-on: self-hosted  # EC2 self-hosted runner

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Install Dependencies
        run: npm install

      - name: Build React App
        run: npm run build

      - name: Copy Build to NGINX
        run: |
          sudo rm -rf /usr/share/nginx/html/*
          sudo cp -a ${{ github.workspace }}/build/. /usr/share/nginx/html/

      - name: Restart NGINX
        run: sudo systemctl restart nginx
```

---

## ⚙️ Configure NGINX to Serve React

```bash
sudo nano /etc/nginx/sites-available/default
```

Update it like this:

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /usr/share/nginx/html;
    index index.html;

    server_name _;

    location / {
        try_files $uri /index.html;
    }
}
```

Then restart NGINX:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

## ✅ Deployment Result

Once the workflow runs successfully, visit:

```
http://<your-ec2-public-ip>
```

You should see your React app instead of the default NGINX page.

---

## 🧼 Troubleshooting

| Issue                        | Fix |
|-----------------------------|-----|
| Still seeing "Welcome to nginx!" | NGINX serving wrong root → Update to `/usr/share/nginx/html` |
| React not loading routes    | Add `try_files $uri /index.html;` in NGINX config |
| GitHub Actions copy step fails | Use absolute path: `${{ github.workspace }}/build/.` |
| Permission denied on restart | Ensure runner user has passwordless sudo or elevate permissions |

---

## 📌 Notes

- You can enable HTTPS with Let's Encrypt (`certbot`) later
- Use `gh-pages` or GitHub-hosted runners for alternative deployment options

---

## 🙌 Author

Maintained by Tirup using AWS EC2, GitHub Actions, and React.
