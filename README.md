# Publish Artifact to Nexus on DigitalOcean

## Overview

Demo project showing how to run Nexus on a DigitalOcean Droplet and publish Java artifacts to it using both Gradle and Maven.

## Tech Stack

![Nexus](https://img.shields.io/badge/Nexus-Repository-blue)
![DigitalOcean](https://img.shields.io/badge/DigitalOcean-Cloud-0080FF)
![Java](https://img.shields.io/badge/Java-8-red)
![Gradle](https://img.shields.io/badge/Gradle-Build-02303A)
![Maven](https://img.shields.io/badge/Maven-Build-C71A36)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-yellow)

## Topics Covered

- Install and configure Nexus from scratch on a cloud server
- Create a Nexus user with relevant permissions
- Build a JAR and upload to Nexus with **Gradle**
- Build a JAR and upload to Nexus with **Maven**

---

## 🚀 Installing & Configuring Nexus on DigitalOcean

### Step 1 — Create a Droplet

Log in to DigitalOcean and create a new Droplet with at least **4 GB RAM** (8 GB recommended). Open port **22** in the firewall for SSH access.

### Step 2 — Install Java & net-tools

```bash
ssh root@<droplet-ip-address>

apt update
apt install openjdk-8-jre-headless
apt install net-tools
```

### Step 3 — Install Nexus

```bash
cd /opt
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -zxvf latest-unix.tar.gz
```

### Step 4 — Create a `nexus` User

```bash
adduser nexus

chown -R nexus:nexus nexus-3.46.0-01
chown -R nexus:nexus sonatype-work
```

### Step 5 — Configure Nexus to Run as `nexus` User

Edit `nexus-3.46.0-01/bin/nexus.rc` and add:
