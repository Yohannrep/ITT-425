```markdown
# Docker and Docker Compose Installation Guide (Ubuntu)

This guide provides a step-by-step process for installing the official Docker Engine and Docker Compose on Ubuntu.

---

## Step 1: Install Docker Engine

First, update your package list and install the necessary dependencies:

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

```

### Add Docker’s Official GPG Key

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

```

### Setup the Official Repository

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
[https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

### Install Docker Packages

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

```

**Verify the installation:**

```bash
docker --version

```

---

## Step 2: Run Docker Without Sudo (Recommended)

To avoid typing `sudo` before every docker command, add your user to the docker group:

```bash
sudo usermod -aG docker $USER
newgrp docker

```

**Test it:**

```bash
docker run hello-world

```

---

## Step 3: Install Docker Compose Standalone

Download the Docker Compose binary (v2.25.0):

```bash
sudo curl -SL [https://github.com/docker/compose/releases/download/v2.25.0/docker-compose-linux-x86_64](https://github.com/docker/compose/releases/download/v2.25.0/docker-compose-linux-x86_64) -o /usr/local/bin/docker-compose

```

### Apply Execute Permissions

```bash
sudo chmod +x /usr/local/bin/docker-compose

```

**Verify the installation:**

```bash
docker-compose version

```

---

*Note: If you are using a non-x86 architecture (like a Raspberry Pi), ensure you download the correct binary for your CPU.*

