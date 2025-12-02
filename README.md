########## SPA Nginx AWS ALB Configuration #######################

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=flat&logo=nginx&logoColor=white)](https://nginx.org/)
[![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=flat&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)

Production-ready Nginx configuration for Single Page Applications (SPA) deployed on AWS with Application Load Balancer (ALB). **Solves the infamous 404 error on page refresh** for React, Vue, Angular, and other client-side routing applications.

## Quick Start

# Clone the repository
git clone https://github.com/yourusername/spa-nginx-aws-alb-config.git

# Copy nginx.conf to your project
cp nginx.conf /path/to/your/project/

# Build Docker image
docker build -t my-spa-app .

1. Global Configuration Block

        user nginx;
        worker_processes auto;
        error_log /var/log/nginx/error.log warn;
        pid /var/run/nginx.pid;

Explanation:

* user nginx: Runs nginx processes under the 'nginx' user for security

* worker_processes auto: Automatically detects CPU cores and creates optimal worker processes

* error_log: Defines where error logs are stored and log level (warn)

* pid: Specifies the process ID file location
    
Why Important: Ensures nginx runs securely and efficiently with proper logging.

2. Events Block

              events {
               worker_connections 1024;
               use epoll;
               multi_accept on;
              }
      
Explanation:

* worker_connections 1024: Each worker can handle 1024 simultaneous connections

* use epoll: Uses efficient epoll event method (Linux-specific)

* multi_accept on: Workers accept multiple connections at once

* Performance Impact: Can handle up to 1024 concurrent users per worker process.
    
3. HTTP Block - MIME Types

        http {
        include /etc/nginx/mime.types;
        default_type application/octet-stream;
        }
        
Purpose: Ensures proper content-type headers for different file types (CSS, JS, images, etc.)

4. Logging Configuration

        log_format main '$remote_addr - $remote_user [$time_local] "$request" '
            '$status $body_bytes_sent "$http_referer" '
            '"$http_user_agent" "$http_x_forwarded_for"';
        access_log /var/log/nginx/access.log main;

What It Logs:

* Client IP address

* Request timestamp

* HTTP request details

* Response status and size

* User agent and referrer information
      
ALB Integration: The $http_x_forwarded_for captures the real client IP behind the load balancer.

5. Performance Optimizations

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        client_max_body_size 16M;
   
Performance Benefits:

* sendfile on: Efficient file serving using kernel sendfile()

* tcp_nopush/tcp_nodelay: Optimizes TCP packet transmission

* keepalive_timeout 65: Keeps connections alive for 65 seconds

* client_max_body_size 16M: Allows file uploads up to 16MB
      
6. Gzip Compression

              gzip on;
              gzip_vary on;
              gzip_min_length 1024;
              gzip_proxied any;
              gzip_comp_level 6;
              gzip_types [various file types];
      
Bandwidth Savings: Compresses responses by 60-80%, significantly reducing load times.

7. Server Block - The Core Solution

        server {
          listen 80;
          server_name localhost;
          root /usr/share/nginx/html;
          index index.html;
           }

Container Setup: Configures nginx to serve files from the standard Docker nginx location.

8. Security Headers

        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Referrer-Policy "no-referrer-when-downgrade" always;

Security Benefits:

* Prevents clickjacking attacks

* Blocks XSS attempts

* Prevents MIME type sniffing

* Controls referrer information
    
9. Static Asset Caching

        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
                expires 1y;
                add_header Cache-Control "public, immutable";
        try_files $uri =404;
        }
    
   Performance Impact: Static assets are cached for 1 year, dramatically reducing server load and improving user experience.

10. The Magic Fix - Client-Side Routing Handler

            location / {
        try_files $uri $uri/ /index.html;
            }
    
  This is the key solution!
  
* First, try to serve the exact file requested ($uri)

* If not found, try as a directory ($uri/)

* If still not found, serve index.html (letting the SPA handle routing)
  
  Why It Works: This ensures that any route refresh falls back to index.html, allowing your SPA's JavaScript router to handle the URL.
  
11. Health Check Endpoint

              location /health 
              {
                  access_log off;
                  return 200 "healthy\n";
                  add_header Content-Type text/plain;
              }
              
ALB Integration: Provides a lightweight endpoint for ALB health checks without cluttering access logs.


