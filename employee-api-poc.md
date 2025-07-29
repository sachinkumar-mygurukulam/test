# Employee API Proof of Concept


---
## Author Information
| Last Updated On | Version | Author       | Level           | Reviewer   |
|-----------------|---------|--------------|-----------------|------------|
| 29-07-2025      | V1.0    | Sachin Kumar | Internal Review | Pritam     |
| 31-07-2025      | V1.1    | Sachin Kumar | L0              |Shreya/Sharvani|
|                 |         | Sachin Kumar | L1              | Abhishek V |
|                 |         | Sachin Kumar | L2              | Abhishek Dubey/Rishabh sharma|
---

## Table of Contents

- [Objective](#objective)
- [Pre-requisites](#pre-requisites)
  - [System Requirements](#system-requirements)
  - [Dependencies](#dependencies)
  - [Important Ports](#important-ports)
- [Step-by-step Installation](#step-by-step-installation)
- [Basic Operations](#basic-operations)
- [Troubleshooting](#troubleshooting)
- [Contact Information](#contact-information)
- [References](#references)

---

## Objective

This Proof of Concept demonstrates the installation, configuration, and basic usage of the Employee REST API—a Golang-based microservice that performs employee-related operations within the OT-Microservices architecture. The API is platform-independent and relies on ScyllaDB (mandatory) as its database and Redis (optional) for caching. This document helps users set up the environment, run the service, and test its endpoints.

---

## Pre-requisites

### System Requirements

| Hardware Specifications | Minimum Recommendation            |
|------------------------ |-----------------------------------|
| Processor              | 2 cores                           |
| RAM                    | 4GB                               |
| Disk                   | 20GB SSD                          |
| OS                     | Ubuntu 22.04 LTS                  |
| Network                | Internet access for package install|

### Dependencies

| Name      | Version      | Description                                     |
|-----------|--------------|-------------------------------------------------|
| Golang    | 1.21.6+      | For building and running the API                |
| ScyllaDB  | 5.4+         | Main database                                   |
| Redis     | Latest       | Optional caching                                |
| Java      | 17           | For ScyllaDB driver/sample scripts              |
| migrate   | Latest       | DB migrations                                   |
| jq        | Latest       | CLI JSON processor                              |

### Important Ports

| Port | Description                         |
|------|-------------------------------------|
| 9042 | ScyllaDB CQL native transport port  |
| 6379 | Redis default port                  |
| 8080 | Employee API default HTTP port      |
| 8081 | Alternative API port (if 8080 busy) |

---

## Step-by-step Installation

### 1. Clone the Employee API Repository

```bash
git clone https://github.com/OT-MICROSERVICES/employee-api.git
cd employee-api
```

### 2. Install ScyllaDB

```bash
sudo mkdir -p /etc/apt/keyrings
sudo gpg --homedir /tmp --no-default-keyring --keyring /etc/apt/keyrings/scylladb.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys d0a112e067426ab2
sudo wget -O /etc/apt/sources.list.d/scylla.list http://downloads.scylladb.com/deb/debian/scylla-5.4.list
sudo apt-get update
sudo apt-get install -y scylla
```

### 3. Install Java 17 (required for ScyllaDB tools)

```bash
sudo apt-get install -y openjdk-17-jre-headless
sudo update-java-alternatives --jre-headless -s java-1.17.0-openjdk-amd64
java -version
```

### 4. Configure ScyllaDB

Edit `/etc/scylla/scylla.yaml`:

```bash
sudo nano /etc/scylla/scylla.yaml
```
- Set these fields with your private IP:
  ```
  seeds: "<PRIVATE_IP>"
  listen_address: "<PRIVATE_IP>"
  rpc_address: "<PRIVATE_IP>"
  ```
- Comment out: `developer_mode = true`

### 5. Initialize and Start ScyllaDB

```bash
sudo scylla_setup
sudo systemctl start scylla-server
sudo systemctl status scylla-server
```

Check node status:

```bash
nodetool status
```

### 6. Create the Employee Database

Enter ScyllaDB shell:

```bash
cqlsh <PRIVATE_IP>
CREATE KEYSPACE IF NOT EXISTS employee_db WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
EXIT
```

### 7. Install Required Utilities

#### DB Migration Tool

```bash
curl -s https://packagecloud.io/install/repositories/golang-migrate/migrate/script.deb.sh | sudo bash
sudo apt-get update
sudo apt-get install -y migrate
migrate -version
```

#### jq (JSON processor)

```bash
sudo apt install jq
jq --version
```

### 8. Install Golang

#### From Official Tarball

```bash
cd
curl -OL https://golang.org/dl/go1.21.6.linux-amd64.tar.gz
sha256sum go1.21.6.linux-amd64.tar.gz
sudo tar -C /usr/local -xvf go1.21.6.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.profile
source ~/.profile
go version
```

#### If Go not working, fix Python error

```bash
sudo apt update
sudo apt install python3-apt -y
sudo apt install golang-go -y
go version
```

### 9. Set Up Employee API

In `employee-api` directory:

```bash
go mod tidy    # Installs dependencies
go run main.go # Starts the API server
```

#### If port 8080 is busy:

Edit `main.go` or `router.go`:

```go
router.Run(":8080")  # Change to another port, e.g. ":8081"
```

### 10. Update ScyllaDB Host in API Config

Edit `config.yaml`:

```yaml
scylladb:
  host: <YOUR_INSTANCE_IP>
  port: 9042
```

### 11. (Optional) Install and Start Redis

```bash
sudo apt-get install redis-server -y
sudo systemctl enable redis-server
sudo systemctl start redis-server
sudo systemctl status redis-server
```

---

## Basic Operations

### 1. Run the Employee API

```bash
go run main.go
```
Or as a background service:

```bash
nohup ./employee-api > output.log 2>&1 &
ps aux | grep employee-api
```

### 2. Health Check

```bash
curl http://localhost:8081/api/v1/employee/health
# Response:
{"message":"Employee API is running fine and ready to serve requests"}
```
Or test in browser:

```
http://<your-public-ip>:8081/api/v1/employee/health
```

### 3. API Endpoints

| Endpoint                             | Method | Description                   |
|-------------------------------------- |--------|-------------------------------|
| /api/v1/employee/health              | GET    | Health check                  |
| /api/v1/employee                     | POST   | Create employee record        |
| /api/v1/employee/{id}                | GET    | Get employee by ID            |
| /api/v1/employee/{id}                | PUT    | Update employee               |
| /api/v1/employee/{id}                | DELETE | Delete employee               |
| /api/v1/employee                     | GET    | List all employees            |

---

## Troubleshooting

- **API not running?**
  - Check for port conflicts:
    ```
    netstat -tuln | grep 8080
    ```
    Change port in code if needed.

- **ScyllaDB issues?**
  - Check service logs:
    ```
    sudo journalctl -u scylla-server
    ```
  - Check DB status:
    ```
    nodetool status
    ```

- **Go not found / Python error?**
  - Fix with:
    ```
    sudo apt update
    sudo apt install python3-apt -y
    sudo apt install golang-go -y
    ```

- **Redis not running?**
  - Start service:
    ```
    sudo systemctl start redis-server
    ```

- **Wrong DB IP in config?**
  - Edit `config.yaml` and update `scylladb.host`.

---

## Contact Information

| Name            | Email address                           |
|-----------------|----------------------------------------|
| Sachin Kumar     | [sachin.kumar.snaatak@mygurukulam.co](sachin.kumar.snaatak@mygurukulam.co)                  |

---

## References

| Link                                                                                   | Description                               |
|---------------------------------------------------------------------------------------|-------------------------------------------|
| [Employee API GitHub](https://github.com/OT-MICROSERVICES/employee-api)               | Employee API source code                  |
| [ScyllaDB Documentation](https://docs.scylladb.com/)                                  | Official ScyllaDB documentation           |
| [ScyllaDB Installation Guide](https://docs.scylladb.com/stable/operating-scylla/procedures/install/install-ubuntu.html) | ScyllaDB Ubuntu installation guide      |
| [Go Documentation](https://golang.org/doc/)                                           | Official Golang docs                      |
| [Redis Documentation](https://redis.io/documentation)                                 | Official Redis docs                       |

