# Google Cloud Platform (GCP) Learning Guide

A comprehensive learning resource for understanding Google Cloud Platform fundamentals, including core concepts, architecture, services, and best practices.

## Table of Contents

- [Core Concepts](#core-concepts)
- [Geographical Setup](#geographical-setup)
- [Resource Hierarchy](#resource-hierarchy)
- [Comparison with AWS](#comparison-with-aws)
- [Billing](#billing)

## Core Concepts

### Projects
Projects are the top-level containers for resources in GCP. Each project has its own set of resources, such as:
- Compute Engine instances
- Cloud Storage buckets
- Cloud SQL databases

### Resource Hierarchy
Resources are organized in a tree-like structure to maintain separation based on departments, business units, use cases, or teams.

### Identity and Access Management (IAM)
IAM is the system that GCP uses to control who has access to resources. It allows you to:
- Create users and groups
- Define roles with specific permissions
- Assign roles to resources

### Networking
GCP provides various networking services to enable resource communication:
- **Cloud VPC**: Virtual Private Cloud for network isolation
- **Cloud Load Balancing**: Distributes traffic across multiple servers
- **Cloud DNS**: Domain name resolution

### Shared VPC
A feature that allows you to share a single VPC network with multiple projects. Useful for organizations that want to centralize network administration.

### Load Balancing
Distributes traffic across multiple servers to improve application performance and reliability.

## Geographical Setup

GCP infrastructure is distributed globally:
- **40 Regions** (geographic locations): Example: `us-west1`
- **Zones** (individual data centers within regions): Examples: `us-west1-a`, `us-west1-b`, `us-west1-c`

## Resource Hierarchy

```
Organization
  ├── Folders
  │   ├── Projects
  │   │   └── Resources
```

## Comparison with AWS

| GCP Concept | AWS Equivalent |
| ----------- | -------------- |
| Region | Region |
| Zone | Availability Zone |
| Instance Group | Target Group |
| Instance Template | Launch Template |
| Cloud Load Balancer | Load Balancer |

## Billing

GCP provides comprehensive billing management features:
- **Billing Alerts**: Set up notifications for spending thresholds
- **Cost Analysis**: Monitor and analyze resource usage costs
- **Budget Controls**: Define budgets and spending limits per project
