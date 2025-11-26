# Gym Management App - DevOps Learning Project

## 🎯 Project Overview

This is a **DevOps demonstration project** created to showcase skills in deployment, containerization, CI/CD, and infrastructure management. Built as part of academic learning at Savitribai Phule Pune University (SPPU).

**Purpose:** Learning and demonstrating DevOps practices for job opportunities in DevOps Engineering.

## 🚀 Tech Stack

### Backend
- Node.js + Express.js
- RESTful API architecture
- MariaDB/MySQL database

### Frontend
- HTML5, CSS3, JavaScript
- Responsive design
- Bootstrap framework

### DevOps Tools
- **Containerization:** Docker
- **Orchestration:** Kubernetes
- **CI/CD:** Jenkins
- **Cloud:** AWS (EC2, RDS)
- **Monitoring:** Prometheus & Grafana
- **Version Control:** Git & GitHub

## 📋 Features

- Member management system
- Trainer scheduling
- Workout tracking
- Membership plans management
- Payment tracking
- Admin dashboard

## 🛠️ Deployment Instructions

### Prerequisites
```bash
# Ubuntu Linux (20.04/22.04)
sudo apt update
sudo apt upgrade -y
```

### Method 1: Direct Deployment on Ubuntu

#### Step 1: Install Node.js
```bash
# Install Node.js 18.x LTS
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Verify installation
node --version
npm --version
```

#### Step 2: Install MariaDB
```bash
# Install MariaDB
sudo apt install -y mariadb-server

# Secure installation
sudo mysql_secure_installation

# Login to MariaDB
sudo mysql -u root -p
```

#### Step 3: Create Database
```sql
CREATE DATABASE gym_management;
CREATE USER 'gymadmin'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON gym_management.* TO 'gymadmin'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

#### Step 4: Clone & Setup Project
```bash
# Clone repository
git clone https://github.com/divyal27/gym-management-app.git
cd gym-management-app

# Install dependencies
npm install

# Create .env file
cat > .env << EOF
PORT=3000
DB_HOST=localhost
DB_USER=gymadmin
DB_PASSWORD=your_password
DB_NAME=gym_management
EOF

# Start application
npm start
```

#### Step 5: Setup as System Service
```bash
# Create systemd service
sudo nano /etc/systemd/system/gym-app.service
```

Add this content:
```ini
[Unit]
Description=Gym Management Application
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/gym-management-app
ExecStart=/usr/bin/node /home/ubuntu/gym-management-app/server.js
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable gym-app
sudo systemctl start gym-app
sudo systemctl status gym-app
```

### Method 2: Docker Deployment

#### Step 1: Install Docker
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
```

#### Step 2: Create Dockerfile
```dockerfile
# Create Dockerfile in project root
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

#### Step 3: Create docker-compose.yml
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
      - DB_USER=gymadmin
      - DB_PASSWORD=your_password
      - DB_NAME=gym_management
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: mariadb:10.11
    environment:
      - MYSQL_ROOT_PASSWORD=root_password
      - MYSQL_DATABASE=gym_management
      - MYSQL_USER=gymadmin
      - MYSQL_PASSWORD=your_password
    volumes:
      - db_data:/var/lib/mysql
    restart: unless-stopped

volumes:
  db_data:
```

#### Step 4: Deploy with Docker Compose
```bash
# Install Docker Compose
sudo apt install -y docker-compose

# Start services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f
```

### Method 3: Kubernetes Deployment

#### Step 1: Install Minikube (for testing)
```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start Minikube
minikube start --driver=docker
```

#### Step 2: Create Kubernetes Manifests

**deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gym-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: gym-app
  template:
    metadata:
      labels:
        app: gym-app
    spec:
      containers:
      - name: gym-app
        image: divyal27/gym-management-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: DB_HOST
          value: "mariadb-service"
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
---
apiVersion: v1
kind: Service
metadata:
  name: gym-app-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: gym-app
```

#### Step 3: Deploy to Kubernetes
```bash
# Create secrets
kubectl create secret generic db-secret \
  --from-literal=username=gymadmin \
  --from-literal=password=your_password

# Apply deployment
kubectl apply -f deployment.yaml

# Check status
kubectl get pods
kubectl get services

# Access application
minikube service gym-app-service
```

### Setup Nginx Reverse Proxy

```bash
# Install Nginx
sudo apt install -y nginx

# Create Nginx config
sudo nano /etc/nginx/sites-available/gym-app
```

Add this configuration:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable and restart:
```bash
sudo ln -s /etc/nginx/sites-available/gym-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Setup SSL with Let's Encrypt

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d your-domain.com

# Auto-renewal
sudo certbot renew --dry-run
```

## 🔧 CI/CD with Jenkins

### Jenkinsfile
```groovy
pipeline {
    agent any
    
    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/divyal27/gym-management-app.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Docker Build') {
            steps {
                sh 'docker build -t divyal27/gym-management-app:${BUILD_NUMBER} .'
            }
        }
        
        stage('Docker Push') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub', url: '']) {
                    sh 'docker push divyal27/gym-management-app:${BUILD_NUMBER}'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    kubectl set image deployment/gym-app \
                    gym-app=divyal27/gym-management-app:${BUILD_NUMBER}
                '''
            }
        }
    }
}
```

## 📊 Monitoring Setup

### Install Prometheus
```bash
# Using Docker
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

### Install Grafana
```bash
# Using Docker
docker run -d \
  --name grafana \
  -p 3001:3000 \
  grafana/grafana
```

## 🔒 Security Best Practices

1. **Firewall Configuration**
```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

2. **Environment Variables**: Never commit `.env` files
3. **Database Security**: Use strong passwords
4. **Regular Updates**: Keep system and packages updated
5. **SSH Keys**: Use key-based authentication

## 📝 Project Structure

```
gym-management-app/
├── backend/
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   └── server.js
├── frontend/
│   ├── css/
│   ├── js/
│   └── index.html
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── .dockerignore
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
├── package.json
└── README.md
```

## 🎓 Learning Objectives

- ✅ Containerization with Docker
- ✅ Container orchestration with Kubernetes
- ✅ CI/CD pipeline implementation
- ✅ Cloud deployment (AWS)
- ✅ Infrastructure as Code
- ✅ Monitoring and logging
- ✅ Security best practices
- ✅ Linux system administration

## 🔗 Useful Commands

```bash
# Check application logs
sudo journalctl -u gym-app -f

# Check Docker logs
docker logs -f <container_id>

# Database backup
mysqldump -u gymadmin -p gym_management > backup.sql

# Restore database
mysql -u gymadmin -p gym_management < backup.sql

# Monitor system resources
htop

# Check port usage
sudo netstat -tulpn | grep :3000
```

## 📫 Contact

**Developer:** Divyal Padalkar
**University:** Savitribai Phule Pune University  
**Purpose:** DevOps Learning & Job Portfolio  
**GitHub:** [github.com/divyal27](https://github.com/divyal27)

## 📄 License

This project is created for educational and demonstration purposes.

---

**Note:** This is a learning project to demonstrate DevOps skills. The application structure will be built incrementally as part of the learning process.
