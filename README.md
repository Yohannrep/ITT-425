# ITT-425 Security Operations Center (SOC) Lab

This repository contains Docker Compose deployments for SOC tools used in **ITT-425**.  
Students will deploy services using Docker Compose on a **fresh Ubuntu virtual machine running in Proxmox**.

---

## Included Tools

- **Shuffle** – Security Orchestration, Automation, and Response (SOAR)
- **TheHive** – Incident Response and Case Management  
  - Includes **Cortex** for observable analysis and automated responders (deployed within the same Docker Compose file)

Each tool is deployed using its own `docker-compose.yml` file.

---

## Prerequisites

Before deploying any SOC tools, you **must** install Docker and Docker Compose on your Ubuntu VM.

➡️ **Complete this first:**  
[Docker & Docker Compose Installation Guide](docker-compose-install.md)

> Do not attempt to run any `docker-compose.yml` files until prerequisites are completed.

---

## Repository Structure

```text
ITT-425/
├── README.md
├── docker-compose-install.md
├── shuffle/
│   └── docker-compose.yml
└── thehive/
    └── docker-compose.yml
