# AWS Three-Tier Architecture

## Table of Contents

- [Introduction](#introduction)
- [Architecture Diagram](#architecture-diagram)
- [Architecture Overview](#architecture-overview)
- [Web Tier (Presentation Layer)](#web-tier-presentation-layer)
- [Application Tier (Logic Layer)](#application-tier-logic-layer)
- [Database Tier (Data Layer)](#database-tier-data-layer)
- [Network Design](#network-design)
- [Security](#security)
- [Benefits](#benefits)
- [Technologies Used](#technologies-used)

## Introduction

Three-tier architecture is a software design pattern that separates an application into three logical layers: the presentation tier, the application tier, and the data tier. Each tier runs on its own infrastructure and can be developed, scaled, and maintained independently.

This document describes a three-tier architecture implemented on Amazon Web Services (AWS), designed for high availability, security, and scalability.

## Architecture Diagram

![AWS Three-Tier Architecture] ./images/three tire architecture.png

*Figure 1: AWS Three-Tier Architecture showing web, application, and database tiers deployed across multiple Availability Zones within a VPC.*

## Architecture Overview

The architecture consists of three distinct tiers deployed within a single AWS VPC:

1. **Web Tier (Presentation Layer)** - Handles incoming user requests and serves the frontend. Deployed in public subnets.
2. **Application Tier (Logic Layer)** - Processes business logic and API requests. Deployed in private subnets.
3. **Database Tier (Data Layer)** - Stores and manages persistent data. Deployed in isolated private subnets.

Traffic flows from users through an Internet Gateway to the Web Tier via an Application Load Balancer. The Web Tier forwards API requests to the Application Tier through an Internal Load Balancer. The Application Tier reads and writes data to the Database Tier. Each tier communicates only with its adjacent tiers, enforced by security groups.

## Web Tier (Presentation Layer)

The Web Tier is the only layer directly accessible from the internet. It receives user requests and serves frontend content.

**AWS Components:**

- **Internet Gateway** - Provides internet access to the VPC
- **Application Load Balancer (ALB)** - Distributes incoming HTTP/HTTPS traffic across web servers
- **EC2 Instances (Auto Scaling Group)** - Hosts web servers (e.g., Nginx serving a React or static site) in public subnets across two Availability Zones
- **NAT Gateway** - Not required for this tier (instances have public IPs via the Internet Gateway)

**Responsibilities:**

- Accept and terminate HTTPS connections
- Serve static frontend content
- Forward API calls to the Application Tier via the Internal Load Balancer

## Application Tier (Logic Layer)

The Application Tier contains the backend business logic. It runs in private subnets with no direct internet access.

**AWS Components:**

- **Internal Application Load Balancer** - Routes requests from the Web Tier to application servers
- **EC2 Instances (Auto Scaling Group)** - Hosts backend application code (e.g., Node.js, Python, Java) in private subnets across two Availability Zones
- **NAT Gateway** - Allows outbound internet access for software updates and external API calls

**Responsibilities:**

- Process business logic and validate data
- Handle authentication and authorization
- Communicate with the Database Tier for data persistence
- Call external APIs when needed (via NAT Gateway)

## Database Tier (Data Layer)

The Database Tier stores all persistent application data. It runs in isolated private subnets accessible only from the Application Tier.

**AWS Components:**

- **Amazon RDS (MySQL or PostgreSQL)** - Managed relational database service
- **Multi-AZ Deployment** - Automatic failover to a standby instance in a different Availability Zone
- **DB Subnet Group** - Ensures the database is placed in designated private subnets

**Responsibilities:**

- Store and manage application data
- Handle read/write queries from the Application Tier
- Provide automatic backups and point-in-time recovery
- Ensure high availability through Multi-AZ failover

## Network Design

The entire architecture is deployed within a single VPC with a CIDR block (e.g., 10.0.0.0/16).

**Subnet Layout:**

- **Public Subnets** (2x, one per AZ) - Host the Web Tier, Internet Gateway routes, and NAT Gateways
- **Private Subnets** (2x, one per AZ) - Host the Application Tier with route to NAT Gateway for outbound traffic
- **Isolated Private Subnets** (2x, one per AZ) - Host the Database Tier with no internet route

**Routing:**

- Public subnets route 0.0.0.0/0 traffic to the Internet Gateway
- Private subnets route 0.0.0.0/0 traffic to the NAT Gateway
- Isolated subnets have no route to the internet

## Security

Security is enforced through a layered security group strategy following the principle of least privilege.

**Security Group Rules:**

- **Web Tier SG** - Inbound: Allow HTTP (80) and HTTPS (443) from 0.0.0.0/0. Outbound: Allow traffic to App Tier SG.
- **App Tier SG** - Inbound: Allow traffic on the application port (e.g., 3000 or 8080) from Web Tier SG only. Outbound: Allow traffic to DB Tier SG and to 0.0.0.0/0 via NAT Gateway.
- **Database Tier SG** - Inbound: Allow MySQL (3306) or PostgreSQL (5432) from App Tier SG only. Outbound: None required.

This chain ensures that the database is never directly accessible from the internet or even from the web tier. Each layer can only communicate with its immediate neighbor.

## Benefits

- **Scalability** - Each tier can be scaled independently based on its specific load. The web tier can scale separately from the application tier.
- **Security** - Network isolation ensures that a compromise in one tier does not automatically grant access to other tiers.
- **High Availability** - Multi-AZ deployment across all tiers protects against single Availability Zone failures.
- **Maintainability** - Separation of concerns allows teams to update one tier without affecting others.
- **Fault Isolation** - Issues in the application logic do not directly impact the database or frontend.

## Technologies Used

- Amazon VPC
- Internet Gateway
- NAT Gateway
- Application Load Balancer (ALB)
- Amazon EC2 with Auto Scaling Groups
- Amazon RDS (MySQL/PostgreSQL)
- Security Groups
- Multiple Availability Zones
