
# 🏆 **Deployment and Maintenance Guide**  
### 🖥️ **Project:** Translation API and UI  
### 📅 **Last Updated:** March 12, 2025  

## 📌 **Project Overview**
| Component | Description |
|-----------|-------------|
| **API** | Flask-based translation API |
| **UI** | NextJS-based translation app |
| **Database** | PostgreSQL |
| **Environment** | Ubuntu 24.04 |
| **Ports** | API → `5000`, UI → `3000` |
| **Process Manager** | PM2 (for UI) |
| **Web Server** | Gunicorn (for Flask API) |

## 🚀 **1. Start the Flask API**
### ✅ **SSH into the VM:**
```bash
ssh ubuntu@<VM_IP>
```

### ✅ **Navigate to the API Directory:**
```bash
cd /opt/api/flask-app
```

### ✅ **Activate the Virtual Environment:**
```bash
source venv/bin/activate
```

### ✅ **Start Gunicorn (API):**
```bash
gunicorn --bind 0.0.0.0:5000 app:app -D
```

### ✅ **Check If Flask API Is Running:**
```bash
sudo ss -tuln | grep 5000
```

### ✅ **Test API Endpoint:**
```bash
curl -X POST http://localhost:5000/predict \
-H "Content-Type: application/json" \
-d '{
  "instances": [
    {"eng": "Hello"}
  ]
}'
```

## 🌐 **2. Start the NextJS UI**
### ✅ **Navigate to the Project Directory:**
```bash
cd /opt/nitm-translate
```

### ✅ **Start UI Using PM2:**
```bash
pm2 start npm --name "nitm-translate" -- start
```

### ✅ **Check UI Status:**
```bash
pm2 list
```

### ✅ **Test UI:**
- Open in browser:
```
http://<VM_IP>:3000
```

## 🔥 **3. Firewall Settings**
### ✅ **Enable Firewall:**
```bash
sudo ufw enable
```

### ✅ **Allow API and UI Ports:**
```bash
sudo ufw allow 5000
sudo ufw allow 3000
```

### ✅ **Check Firewall Status:**
```bash
sudo ufw status
```

## 🎯 **4. Database Configuration**
### ✅ **Connect to PostgreSQL:**
```bash
sudo -u postgres psql
```

### ✅ **List All Databases:**
```sql
\l
```

### ✅ **Connect to Database:**
```sql
\c feedback_db;
```

### ✅ **List All Tables:**
```sql
\dt;
```

### ✅ **Grant Permissions to API User:**
If you see "permission denied" errors:
```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO feedback_user;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO feedback_user;
```

## 🛡️ **5. Start Services After Reboot**
### ✅ **Start Flask API:**
```bash
cd /opt/api/flask-app
source venv/bin/activate
gunicorn --bind 0.0.0.0:5000 app:app -D
```

### ✅ **Start NextJS UI:**
```bash
cd /opt/nitm-translate
pm2 start npm --name "nitm-translate" -- start
```

### ✅ **Restart PostgreSQL (if needed):**
```bash
sudo systemctl restart postgresql
```

## 🔍 **6. Logs and Debugging**
### ✅ **Flask Logs (Gunicorn):**
```bash
sudo journalctl -u gunicorn -f
```

### ✅ **UI Logs (PM2):**
```bash
pm2 logs nitm-translate
```

### ✅ **PostgreSQL Logs:**
```bash
sudo tail -f /var/log/postgresql/postgresql.log
```

## ✅ **7. Code Update and Restart**
### **To update the code and restart:**
1. Update code:
```bash
git pull origin main
```

2. Restart API:
```bash
sudo systemctl restart gunicorn
```

3. Restart UI:
```bash
pm2 restart nitm-translate
```

## 🌐 **8. Endpoint Summary**
| Component | Endpoint | Example |
|-----------|----------|---------|
| **API** | `/predict` | `http://<VM_IP>:5000/predict` |
| **UI** | `/` | `http://<VM_IP>:3000` |
| **Feedback API (POST)** | `/feedback` | `http://<VM_IP>:5000/feedback` |
| **Feedback API (GET)** | `/feedback` | `http://<VM_IP>:5000/feedback` |

## 🚨 **9. Permissions Fix (If Needed)**
If Gunicorn cannot read the files:
```bash
sudo chown -R ubuntu:www-data /opt/api/flask-app
sudo chmod -R 770 /opt/api/flask-app
```

## 🛠️ **10. Environment Variables (.env)**
The `.env` file should be present at:
```bash
/opt/api/flask-app/.env
```

Example `.env`:
```plaintext
DB_HOST=localhost
DB_PORT=5432
DB_USER=feedback_user
DB_PASSWORD=feedback_password
DB_NAME=feedback_db
```

### ✅ **Load Environment Variables:**
```bash
source .env
```

## 🚀 **11. Automate Startup After Reboot**
### ✅ **Create a Startup Script:**
1. Create a startup script:
```bash
nano ~/startup.sh
```

2. Add:
```bash
#!/bin/bash

# Load environment
cd /opt/api/flask-app
source venv/bin/activate

# Start Flask API
gunicorn --bind 0.0.0.0:5000 app:app -D

# Start NextJS UI
cd /opt/nitm-translate
pm2 start npm --name "nitm-translate" -- start
```

3. Make the script executable:
```bash
chmod +x ~/startup.sh
```

4. Add to crontab:
```bash
crontab -e
```
Add:
```bash
@reboot /home/ubuntu/startup.sh
```

## ✅ **12. Deployment Status:** ✅ SUCCESSFUL ✅
