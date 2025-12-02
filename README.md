########## SPA Nginx AWS ALB Configuration #######################

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=flat&logo=nginx&logoColor=white)](https://nginx.org/)
[![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=flat&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)

Production-ready Nginx configuration for Single Page Applications (SPA) deployed on AWS with Application Load Balancer (ALB). **Solves the infamous 404 error on page refresh** for React, Vue, Angular, and other client-side routing applications.

## Quick Start

```bash
# Clone the repository
git clone https://github.com/yourusername/spa-nginx-aws-alb-config.git

# Copy nginx.conf to your project
cp nginx.conf /path/to/your/project/

# Build Docker image
docker build -t my-spa-app .
