# Employee API POC
<img width="311" height="162" alt="Image" src="https://github.com/user-attachments/assets/4d1f784e-0f45-499b-86f5-47ca6e6c4940" />

## Author Information
| Last Updated On | Version | Author       | Level           | Reviewer   |
|-----------------|---------|--------------|-----------------|------------|
| 29-07-2025      | V1.0    | Sachin Kumar | Internal Review | Pritam     |
| 31-07-2025      | V1.1    | Sachin Kumar | L0              |Shreya/Sharvari|
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

This Proof of Concept demonstrates the installation, configuration, and basic usage of the Employee REST API—a Golang-based microservice that performs employee-related operations within the OT-Microservice.

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

<img width="859" height="296" alt="Image" src="https://github.com/user-attachments/assets/0d8cf109-daa2-4f2c-bc10-cc6761e7aa80" />

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
<img width="758" height="161" alt="Image" src="https://github.com/user-attachments/assets/e0a6688e-4192-4bdb-b3d1-949d07e29a19" />

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

<img width="758" height="481" alt="Image" src="https://github.com/user-attachments/assets/ed85ccd9-ebf0-4342-8ab1-a286f2515f72" />

### 5. Initialize and Start ScyllaDB

```bash
sudo scylla_setup
sudo systemctl start scylla-server
sudo systemctl status scylla-server
```
<img width="1362" height="517" alt="Image" src="https://github.com/user-attachments/assets/c2d94902-0ee0-4d23-b89f-eadfec125300" />

Check node status:

```bash
nodetool status
```

 <img width="756" height="253" alt="Image" src="https://github.com/user-attachments/assets/d67da6c9-44ab-465b-97f3-5244c07aadf6" />

### 6. Create the Employee Database

Enter ScyllaDB shell:

```bash
cqlsh <PRIVATE_IP>
CREATE KEYSPACE IF NOT EXISTS employee_db WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
EXIT
```
<img width="1362" height="517" alt="Image" src="https://github.com/user-attachments/assets/fb95a855-c7c7-424e-8936-5fa0cfd2695f" />

### 7. Install Required Utilities

#### DB Migration Tool

```bash
curl -s https://packagecloud.io/install/repositories/golang-migrate/migrate/script.deb.sh | sudo bash
sudo apt-get update
sudo apt-get install -y migrate
migrate -version
```
<img width="1362" height="517" alt="Image" src="https://github.com/user-attachments/assets/f1bd3264-ca7c-4837-a389-c2a57252638f" />

#### jq (JSON processor)

```bash
sudo apt install jq
jq --version
```

<img width="696" height="517" alt="Image" src="https://github.com/user-attachments/assets/86852ec7-35eb-4cdb-8261-4cb2cbe85d1e" />

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

<img width="711" height="517" alt="Image" src="https://github.com/user-attachments/assets/eb8c9127-a6d1-4383-8275-643230e0207b" />

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

---

### 10. Run Employee API as a Background Service 

To keep your API running even after closing the terminal, you should run it as a background service.
Running your API as a systemd service ensures it will automatically restart on crash and survive server reboots.

1. **Create a systemd service file:**

   ```bash
   sudo nano /etc/systemd/system/employee-api.service
   ```

2. **Paste this configuration (adjust paths if needed):**

   ```ini
   [Unit]
   Description=Employee API Service
   After=network.target

   [Service]
   User=ubuntu
   WorkingDirectory=/home/ubuntu/employee-api
   ExecStart=/usr/local/go/bin/go run main.go
   Restart=always
   RestartSec=10
   Environment="GIN_MODE=release"

   [Install]
   WantedBy=multi-user.target
   ```
   <img width="878" height="299" alt="Image" src="https://github.com/user-attachments/assets/c2dfb9a9-63a7-461c-8a23-aff077636c93" />

3. **Enable and start the service:**

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start employee-api
   sudo systemctl enable employee-api  
   ```

4. **Verify it's running:**

   ```bash
   sudo systemctl status employee-api
   ```

<img width="802" height="431" alt="Image" src="https://github.com/user-attachments/assets/cccaece9-4b9b-4c26-8cfe-8dc905b00944" />

---

### 11. Update ScyllaDB Host in API Config

Edit `config.yaml`:

```yaml
scylladb:
  host: ["172.31.16.8"]
  username: scylladb
  password: password
  keyspace: employee_db

redis:
  enabled: false
  host: 172.31.16.8
  password: password
  database: 0
```
<img width="756" height="231" alt="Image" src="https://github.com/user-attachments/assets/ca52d4d3-558b-488e-96a4-cad69686b2da" />

### 12. (Optional) Install and Start Redis

```bash
sudo apt-get install redis-server -y
sudo systemctl enable redis-server
sudo systemctl start redis-server
sudo systemctl status redis-server
```

<img width="928" height="299" alt="Image" src="https://github.com/user-attachments/assets/e3839618-fb12-4d26-a007-60cbbc39ec53" />

---

## Basic Operations 

### 1. Run the Employee API

```bash
go run main.go
```

### 2. Health Check

```bash
curl http://localhost:8080/api/v1/employee/health
# Response:
{"message":"Employee API is running fine and ready to serve requests"}
```
<img width="1172" height="504" alt="Image" src="https://github.com/user-attachments/assets/b9057cbe-42f7-4d65-931b-a89045a68883" />

Or test in browser:

```
http://<your-public-ip>:8080/api/v1/employee/health
```
<img width="1362" height="764" alt="Image" src="https://github.com/user-attachments/assets/10f91475-1338-4d77-a021-27623c061665" />

---

## Troubleshooting


- **If Swagger UI shows "page not found", update the Swagger docs URL in your `main.go` file to use your EC2 private IP**

```go
url := ginSwagger.URL("http://<PRIVATE_IP>:8080/swagger/doc.json")
```

Replace `<PRIVATE_IP>` with your EC2's private IP.

    
  Example:  
   <img width="801" height="499" alt="Image" src="https://github.com/user-attachments/assets/a82c2434-c775-4cba-9b72-416fa90c0e7c" />

---

## Contact Information

| Name            | Email address                           |
|-----------------|----------------------------------------|
| Sachin Kumar    | [sachin.kumar.snaatak@mygurukulam.co](sachin.kumar.snaatak@mygurukulam.co) |

---

## References

| Link                                                                                   | Description                               |
|---------------------------------------------------------------------------------------|-------------------------------------------|
| [Employee API GitHub](https://github.com/OT-MICROSERVICES/employee-api)               | Employee API source code                  |
| [ScyllaDB Documentation](https://docs.scylladb.com/)                                  | Official ScyllaDB documentation           |
| [ScyllaDB Installation Guide](https://docs.scylladb.com/stable/operating-scylla/procedures/install/install-ubuntu.html) | ScyllaDB Ubuntu installation guide      |
| [Go Documentation](https://golang.org/doc/)                                           | Official Golang docs                      |
| [Redis Documentation](https://redis.io/documentation)                                 | Official Redis docs                       |
