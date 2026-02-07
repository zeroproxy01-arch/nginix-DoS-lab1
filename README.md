# HTTP Traffic Monitoring and DoS Mitigation Lab (NGINX)

## 📌 Overview
This lab demonstrates how to monitor HTTP traffic and mitigate basic Denial-of-Service (DoS) style attacks using **NGINX rate limiting**.  
By generating controlled traffic and applying request limits, we observe how NGINX protects web services from excessive requests.

---

## 🎯 Objectives
- Understand HTTP traffic behavior under load
- Simulate high request rates using Apache Bench (`ab`)
- Implement NGINX rate limiting
- Compare server behavior before and after mitigation

---

## 🧰 Tools & Technologies
- **Operating System:** Ubuntu Linux
- **Web Server:** NGINX
- **Traffic Generator:** Apache Bench (`ab`)
- **Security Concept:** Rate limiting / DoS mitigation

---

## 🏗 Lab Environment
- Ubuntu VM or Linux system
- Localhost or VM IP accessible from browser

---

## 🚀 Lab Steps

step 1️ Install NGINX
sudo apt update
sudo apt install nginx -y

step 2

 Install Apache Bench
sudo apt install apache2-utils -y

step 3

Baseline Traffic Test (Before Mitigation)

Run a traffic test without rate limiting:

ab -n 200 -c 20 http://localhost/


📌 Observation:

Most requests succeed

Server accepts high request bursts

step 4

Enable NGINX Rate Limiting

Edit the NGINX configuration:

sudo nano /etc/nginx/nginx.conf


Inside the http { } block, add:

limit_req_zone $binary_remote_addr zone=one:10m rate=5r/s;


Inside the server { } block, add:

limit_req zone=one burst=10 nodelay;

step 5 

Restart NGINX
sudo nginx -t
sudo systemctl restart nginx

step 6

Traffic Test After Mitigation

Re-run the same test:

ab -n 200 -c 20 http://localhost/

Step 7

Enable a Dedicated Rate-Limit Log
Open your NGINX site configuration file:

sudo nano /etc/nginx/sites-enabled/default
Inside the existing server { } block, add the following line:

Nginx
error_log /var/log/nginx/rate_limit.log notice;
Your configuration should now look similar to this:
server {
    listen 80 default_server;

    limit_req zone=one burst=10 nodelay;
    error_log /var/log/nginx/rate_limit.log notice;

    root /var/www/html;
    index index.html;
}
Save the file and exit.

Step 8

Reload NGINX Configuration
Validate the configuration and reload NGINX (no restart required):

sudo nginx -t
sudo systemctl reload nginx
This applies the logging changes immediately without stopping the server.

Step 9

Simulate High Traffic (Attack Simulation)
Generate concurrent requests to trigger rate limiting:

ab -n 200 -c 20 http://localhost/
This simulates aggressive traffic hitting the web server

Step 10

View Rate-Limited Events
Check the dedicated rate-limit log file:

sudo cat /var/log/nginx/rate_limit.log
Expected log entries will look similar to:


limiting requests, excess: 5.000 by zone "one"

Step 11

Analyze Attacker Behavior (SOC Perspective)
Count the number of blocked requests:

sudo grep "limiting requests" /var/log/nginx/rate_limit.log | wc -l
This shows how many requests were rate-limited.
(Optional – detailed view):

sudo grep "limiting requests" /var/log/nginx/rate_limit.log
This provides raw evidence of abusive traffic and mitigation effectiveness.
