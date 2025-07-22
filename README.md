# 🚀 Two-Tier Flask Web App Deployment using Docker

This project demonstrates how to **deploy a two-tier Flask web application** using Docker. The application consists of:

- A **Flask-based web app** (cloned from an existing repo)
- A **MySQL database**
- Managed using **Docker Compose** for seamless container orchestration

# ⚙️ How I deployed?

## Step 1: Clone the App Code

The original Flask app was cloned from a public GitHub repository.
git clone https://github.com/LondheShubham153/two-tier-flask-app.git
cd two-tier-flask-app

## Step 2: Create a Dockerfile
A Dockerfile was created to define how the Flask application should be containerized. This includes installing dependencies and specifying the command to run the app:-
# Use an official Python runtime as the base image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# install required packages for system
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y gcc default-libmysqlclient-dev pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Copy the requirements file into the container
COPY requirements.txt .

# Install app dependencies
RUN pip install mysqlclient
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Specify the command to run your application
CMD ["python", "app.py"]



## Step 3: Write docker-compose.yml
A docker-compose.yml file was created to define and run the Flask app and the MySQL database as separate services. This included service configuration, environment variables, volumes, health checks, and networking:-

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


## Step 4: Add .gitignore to Exclude MySQL Data
To avoid committing database runtime files, a .gitignore file was created with:
mysql-data/

## Step 5: Build and Run the Containers
Used Docker Compose to build and run both containers:
docker-compose up --build

## Step 6: Access the Application
Once running, the Flask app was accessible.
It successfully connected to the MySQL database container.

##  Step 7: Stopping and Cleaning Up
To stop the containers:
docker-compose down
