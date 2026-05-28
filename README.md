Log in, change the password, then log in again.

### Step 2 — Create a New User

1. Go to **Settings → Security → Users**
2. Click **Create local user**
3. Set username to `nx-user` and password to `nx-user-pwd`
4. Temporarily assign the `nx-anonymous` role

### Step 3 — Create a Role

1. Go to **Settings → Security → Roles → Create Role**
2. Choose type **Nexus role**
3. Enter an ID and name
4. Add the privilege `nx-repository-view-maven2-*-*` and save

### Step 4 — Assign Role to User

Open the `nx-user` user, assign the new role, and remove `nx-anonymous`.

---

## 🐘 Gradle — Build & Publish to Nexus

### Step 1 — Update `build.gradle`

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

### Step 2 — Add `gradle.properties`

```properties
repoUser=nx-user
repoPassword=nx-user-pwd
```

### Step 3 — Build & Publish

```bash
./gradlew build
./gradlew publish
```

---

## 🔴 Maven — Build & Publish to Nexus

### Step 1 — Update `pom.xml`

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

### Step 2 — Add Credentials to `~/.m2/settings.xml`

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

### Step 3 — Build & Publish

```bash
mvn package
mvn deploy
```

> **Note:** Maven automatically routes to the snapshots or releases repository based on whether the version string ends with `-SNAPSHOT`.
