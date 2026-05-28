# Publish Artifact to Nexus on DigitalOcean

## Overview

Demo project demonstrating how to run a Nexus repository on a DigitalOcean Droplet and publish Java build artifacts to it using both Gradle and Maven.

## Technologies Used

![Nexus](https://img.shields.io/badge/Nexus-Repository-blue)
![DigitalOcean](https://img.shields.io/badge/DigitalOcean-Cloud-0080FF)
![Java](https://img.shields.io/badge/Java-8-orange)
![Gradle](https://img.shields.io/badge/Build-Gradle-02303A)
![Maven](https://img.shields.io/badge/Build-Maven-C71A36)
![Linux](https://img.shields.io/badge/OS-Linux-yellow)

## Table of Contents

- [Install & Configure Nexus on a Cloud Server](#install--configure-nexus-on-a-cloud-server)
- [Create a Nexus User with Permissions](#create-a-nexus-user-with-permissions)
- [Build & Publish with Gradle](#build--publish-with-gradle)
- [Build & Publish with Maven](#build--publish-with-maven)

---

## Install & Configure Nexus on a Cloud Server

### Step 1: Create a Droplet on DigitalOcean

Log in to DigitalOcean and create a new Droplet with **at least 4GB RAM** (8GB recommended). Add a firewall rule to open **port 22** for SSH.

### Step 2: Install Java and net-tools

```bash
# SSH into the server
ssh root@<droplet-ip-address>

# Install Java 8 and net-tools
apt update
apt install openjdk-8-jre-headless
apt install net-tools
```

### Step 3: Install Nexus

```bash
# Download and unpack the latest Nexus version into /opt
cd /opt
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -zxvf latest-unix.tar.gz
```

### Step 4: Create a Nexus User

```bash
adduser nexus

# Grant nexus user access to the unpacked folders
chown -R nexus:nexus nexus-3.46.0-01
chown -R nexus:nexus sonatype-work
```

### Step 5: Configure Nexus to Run as the nexus User

Edit `nexus-3.46.0-01/bin/nexus.rc` and add:

```
run_as_user="nexus"
```

### Step 6: Start Nexus

```bash
# Switch to nexus user and start the service
su - nexus
/opt/nexus-3.46.0-01/bin/nexus start

# Verify the process and port
ps aux | grep nexus       # shows the Nexus PID
netstat -tlnp             # confirms Nexus is listening on port 8081
```

> In DigitalOcean, add a firewall rule to open **port 8081** for all IP addresses.

---

## Create a Nexus User with Permissions

### Step 1: Login as Admin

Navigate to `http://<droplet-ip-address>:8081`. The default admin password is stored at:

```
/opt/sonatype-work/nexus3/admin.password
```

Log in, change the password, then log in again.

### Step 2: Create a New User

1. Click the **Settings** icon
2. Go to **Security → Users**
3. Click **Create local user**
4. Set **ID** to `nx-user`, **password** to `nx-user-pwd`, and assign the `nx-anonymous` role for now

### Step 3: Create a Role

1. Go to **Security → Roles**
2. Click **Create role** → choose type **Nexus role**
3. Enter an ID and name
4. Add the privilege `nx-repository-view-maven2-*-*` and save

### Step 4: Assign Role to User

1. Go back to **Users** and open `nx-user`
2. Assign the new role and remove `nx-anonymous`

---

## Build & Publish with Gradle

### Step 1: Update `build.gradle`

```groovy
plugins {
    id 'maven-publish'
}

version '1.0.0-SNAPSHOT'

publishing {
    publications {
        mavenJava(MavenPublication) {
            artifact("build/libs/java-gradle-app-$version" + ".jar") {
                extension 'jar'
            }
        }
    }
    repositories {
        maven {
            name 'nexus'
            def releasesRepoUrl  = 'http://<nexus-ip-address>:8081/repository/maven-releases/'
            def snapshotsRepoUrl = 'http://<nexus-ip-address>:8081/repository/maven-snapshots/'
            url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
            allowInsecureProtocol = true
            credentials {
                username project.repoUser
                password project.repoPassword
            }
        }
    }
}
```

### Step 2: Add `gradle.properties`

```properties
repoUser=nx-user
repoPassword=nx-user-pwd
```

### Step 3: Build and Publish

```bash
./gradlew build
./gradlew publish
```

---

## Build & Publish with Maven

### Step 1: Update `pom.xml`

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-deploy-plugin</artifactId>
            <version>2.7</version>
        </plugin>
    </plugins>
</build>

<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <url>http://[nexus-ip-address]:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>http://[nexus-ip-address]:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

### Step 2: Add Credentials to `~/.m2/settings.xml`

```xml
<settings>
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>nx-user</username>
            <password>nx-user-pwd</password>
        </server>
        <server>
            <id>nexus-snapshots</id>
            <username>nx-user</username>
            <password>nx-user-pwd</password>
        </server>
    </servers>
</settings>
```

### Step 3: Build and Publish

```bash
mvn package
mvn deploy
```

> Maven automatically deploys to the **snapshot** or **release** repository based on whether the version contains `SNAPSHOT`.
