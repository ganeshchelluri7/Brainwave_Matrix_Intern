# DevConnector - Cloud Deployment Setup

## Overview

**DevConnector** is a full-stack application using **React** (frontend) and **Django** with **Django Rest Framework** (backend). This application is deployed on **AWS EC2** with **Nginx** as the reverse proxy. The frontend is served on **Port 80**, while the backend API runs on **Port 82**. 

### Technologies Used:
- **Frontend**: React.js
- **Backend**: Django, Django Rest Framework (DRF)
- **Cloud Provider**: AWS EC2
- **Reverse Proxy**: Nginx
- **IP Management**: AWS Elastic IP

---

## Cloud Infrastructure Setup

1. **AWS EC2 Instance**:
   - **Instance Type**: t2.micro (Ubuntu 24)
   - **Elastic IP**: `13.203.129.130` (use this for accessing both frontend and backend)
   - **Security Groups**: 
     - SSH (Port 22)
     - HTTP (Port 80)
     - Backend API (Port 82)

2. **Nginx**:
   - Configured to serve **React app** via Port 80.
   - Configured to forward requests to the **Django backend** on Port 82.
   - Reverse proxy for handling all requests efficiently.

---

## Setup Instructions

### 1. **Configure EC2 Instance**:
   - Launch EC2 instance with Ubuntu 24.
   - Assign the **Elastic IP** to the instance.
   - Open necessary ports on the Security Group (22, 80, 82).

### 2. **Install Nginx on EC2**:
   - SSH into EC2: 
     ```bash
     ssh -i <your-key.pem> ubuntu@13.203.129.130
     ```
   - Install Nginx:
     ```bash
     sudo apt update
     sudo apt install nginx
     sudo systemctl start nginx
     sudo systemctl enable nginx
     ```

### 3. **Configure Nginx**:
   - Edit the Nginx configuration file:
     ```bash
     sudo nano /etc/nginx/sites-available/devconnector
     ```
   - Add the following server block:
     ```nginx
     server {
         listen 80;
         server_name 13.203.129.130;

         location / {
             proxy_pass http://127.0.0.1:3000;  # React
         }

         location /api/ {
             proxy_pass http://127.0.0.1:8000;  # Django API
         }
     }
     ```

   - Enable and restart Nginx:
     ```bash
     sudo ln -s /etc/nginx/sites-available/devconnector /etc/nginx/sites-enabled/
     sudo systemctl restart nginx
     ```

### 4. **Deploy Frontend**:
   - Build React app:
     ```bash
     npm run build
     ```
   - Copy the build files to the EC2 instance:
     ```bash
     scp -r build/* ubuntu@13.203.129.130:/var/www/html/
     ```

### 5. **Deploy Backend (Django)**:
   - Install backend dependencies:
     ```bash
     sudo apt install python3-pip python3-dev libpq-dev
     pip install django djangorestframework
     ```
   - Run Django server:
     ```bash
     python3 manage.py runserver 0.0.0.0:8000
     ```

   - (Optional: Use Gunicorn for production)
     ```bash
     pip install gunicorn
     gunicorn --workers 3 projectname.wsgi:application
     ```

---

## Testing the Deployment

1. Visit the frontend at `http://13.203.129.130`.
2. Access the backend API at `http://13.203.129.130/api/`.

If everything is set up correctly, both frontend and backend should be working smoothly.

---

## Repository Structure

Brainwave_Matrix_Intern/
│
├── backend/                         # Django backend files
│   ├── db.sqlite3                   # SQLite database file (if using SQLite)
│   ├── devconnector/                # Django project-specific configurations
│   ├── manage.py                    # Django entry point for managing the project
│   ├── mysite/                      # Additional Django configurations
│   ├── nohup.out                    # Output from the nohup command (server logs)
│   ├── requirements.txt             # List of backend dependencies
│   
│
├── frontend/                        # React frontend files
│   ├── build/                       # React production build files (after `npm run build`)
│   ├── node_modules/                # Node modules (dependencies installed via npm)
│   ├── package-lock.json            # Lock file with specific versions of frontend dependencies
│   ├── package.json                 # Manifest for managing React app dependencies
│   ├── public/                      # Public files for React (including index.html, static assets)
│   └── src/                         # Source files for React app (JSX components, hooks, etc.)
│
├── nginx/                           # Nginx configuration directory
│   ├── front.conf                   # Nginx configuration for the frontend (React app)
│   ├── back.conf                    # Nginx configuration for the backend (Django API)
│
├── env/                             # Python virtual environment
├── LICENSE                          # Project license
├── README.md                        # Project documentation, setup, and deployment instructions
└── .gitignore                       # Defines files/folders to ignore by Git
