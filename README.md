# 🚀 Two-Tier Flask Web App Deployment using Docker

This project demonstrates how to **deploy a two-tier Flask web application** using Docker. The application consists of:

- A **Flask-based web app** (cloned from an existing repo)
- A **MySQL database**
- Managed using **Docker Compose** for seamless container orchestration

---

# ⚙️ How I Deployed It

## 🧩 Step 1: Clone the App Code

The original Flask app was cloned from a public GitHub repository:

git clone https://github.com/LondheShubham153/two-tier-flask-app.git

cd two-tier-flask-app

🐳 Step 2: Create a Dockerfile
A Dockerfile was created to define how the Flask application should be containerized. It installs system and Python dependencies and runs the app.

FROM python:3.9-slim

WORKDIR /app

RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install mysqlclient
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]

🧬 Step 3: Write docker-compose.yml
A docker-compose.yml file was created to define and run the Flask app and the MySQL database as separate services. It includes service configuration, environment variables, health checks, and volumes.

version: "3.8"


services:

  mysql:
	
    user: "${UID}:${GID}" 
    image: mysql:5.7
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: devops
      MYSQL_USER: admin
      MYSQL_PASSWORD: admin
    volumes:
      - ./mysql-data:/var/lib/mysql
      - ./message.sql:/docker-entrypoint-initdb.d/message.sql  
    networks:
      - twotier
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-proot"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 60s
			

  flask-app:
	
    image: trainwithshubham/two-tier-flask-app:latest
    container_name: flask-app
    ports:
      - "5000:5000"
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: root
      MYSQL_DB: devops
    depends_on:
      - mysql
    networks:
      - twotier
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:5000/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
			

networks:
  twotier:
	

📂 Step 4: Add .gitignore to Exclude MySQL Data

To avoid committing MySQL runtime files, a .gitignore file was created with:

mysql-data/


🛠️ Step 5: Build and Run the Containers

Run the following command to build and start both containers:

docker-compose up --build


🌐 Step 6: Access the Application

Once running, access the Flask app in your browser.


🧹 Step 7: Stop and Clean Up

To stop and remove containers:

docker-compose down

To also remove named volumes:

docker-compose down -v



