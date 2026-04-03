# AWS 3-Tier Highly Available Web Application

## Project Overview

This project demonstrates a highly available 3-tier web application architecture deployed on AWS using scalable and fault-tolerant services.

The solution includes:

* **S3 Static Frontend Hosting**
* **Application Load Balancer (ALB)** for path-based routing
* **Network Load Balancer (NLB)** for static IP and Layer 4 routing theory
* **Gateway Load Balancer (GWLB)** for firewall appliance integration
* **EC2 Backend API Fleet** in an Auto Scaling Group
* **AWS Lambda** for authentication service
* **CloudWatch + Step Scaling Policies**

---

## Architecture Components

### 1. S3 Frontend

The frontend is hosted on an S3 bucket configured for static website hosting.

Features:

* Static website hosting enabled
* Public read access for website files
* CORS enabled for API calls
* Integrated with backend APIs through ALB

---

### 2. Application Load Balancer (ALB)

The ALB handles Layer 7 routing and forwards requests based on URL paths.

### Listener Rules

* `/users` → EC2 backend target group
* `/auth` → Lambda target group

### Features

* Path-based routing
* Sticky sessions enabled
* Cross-zone load balancing enabled

---

### 3. Network Load Balancer (NLB) – Theory

The NLB is included as an architectural component for static IP requirements.

### Use Cases

* Static Elastic IP per Availability Zone
* TCP/UDP/TLS traffic handling
* Low-latency Layer 4 routing
* Firewall allowlisting

---

### 4. Gateway Load Balancer (GWLB) – Theory

GWLB is used for routing traffic through security appliances.

### Use Cases

* Firewall appliance insertion
* Deep packet inspection
* Traffic monitoring
* GENEVE protocol support

Traffic Flow:

`Internet → NLB → GWLB → ALB → Backend Services`

---

### 5. EC2 Backend API Fleet

The backend API is hosted on EC2 instances.

### Technology Used

* Python Flask
* Port `5000`
* CORS enabled

### API Endpoints

* `/` → Health check
* `/users` → Returns sample user data

---

### 6. AWS Lambda Authentication Service

The Lambda function handles authentication requests.

### Endpoint

* `/auth`

### Response

Returns:

`Authentication Successful`

---

### 7. Auto Scaling Group (ASG)

The backend instances are managed through an ASG.

### Scaling Policy

* **Scale Out:** `+2 instances` when CPU > `70%`
* **Scale In:** `-1 instance` when CPU < `25%`

### Placement Strategy

* Spread Placement Group
* Instances distributed across different hardware racks

---

## Validation Criteria

* Frontend loads from S3
* `/users` successfully routes to EC2
* `/auth` successfully routes to Lambda
* Auto Scaling triggers on CPU stress
* Instances launch across multiple racks
* Traffic inspection possible through GWLB theory layer

---

## Testing Steps

### Backend Test

```bash
curl http://<alb-dns>/users
```

### Authentication Test

```bash
curl http://<alb-dns>/auth
```

### Stress Test

```bash
stress --cpu 2 --timeout 300
```

---

## Key AWS Services Used

* Amazon S3
* EC2
* Auto Scaling Group
* Application Load Balancer
* Network Load Balancer
* Gateway Load Balancer
* AWS Lambda
* CloudWatch

---

## Outcome

This project demonstrates a production-style scalable AWS architecture with frontend hosting, backend routing, scaling, and security inspection layers.
