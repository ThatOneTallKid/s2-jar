# Setting Up Github Actins for CI/CD

GitHub Actions makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue tracking work the way you want.

## Checklist

- [x] Setup GitHub Actions
- [ ] Setup local 
- [ ] Build project
- [ ] Publish Jar


## Setting up our project for Github Actions

### **1. mvn folder for local configurations**
A personalized maven project needs its own parent `settings.xml` file loacted at the `.m2` folder as such to facilitate publishing and packaging process.

So we will need to add a `.mvn` folder in this case inside the project folder, `s2-app` and `s2-core` in this case.

*Contents of `.mvn` folder*
```
~ .mvn
--maven.config
--local-settings.xml
```

The `.mvn/maven.config` file must specify the local   `settings.xml` file for the project to use it during further processes.

### **maven.config**
```
--settings ./.mvn/local-settings.xml
```

<br/>

### **locall-settings.xml**

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
​
    <!-- this m2 settings file is for internal developers, please replace your /Users/yourlogin/.m2/settings.xml with this file -->
​
    <pluginGroups>
        <pluginGroup>com.numericalmethod</pluginGroup>
    </pluginGroups>
​
    <profiles>
        <profile>
            <id>default</id>
            <repositories>
                <repository>
                    <id>nm-repo</id>
                    <name>Numerical Method Repositories</name>
                    <url>http://repo.numericalmethod.com/maven/</url>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                        <checksumPolicy>warn</checksumPolicy>
                    </snapshots>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>daily</updatePolicy>
                        <checksumPolicy>fail</checksumPolicy>
                    </releases>
                </repository>
                <repository>
                    <id>nm-internal</id>
                    <name>Numerical Method Repositories</name>
                    <url>http://repo.numericalmethod.com:8080/artifactory/internal/</url>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                        <checksumPolicy>warn</checksumPolicy>
                    </snapshots>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>daily</updatePolicy>
                        <checksumPolicy>fail</checksumPolicy>
                    </releases>
                </repository>
			</repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>nm-repo</id>
                    <name>Numerical Method Repositories</name>
                    <url>http://repo.numericalmethod.com/maven/</url>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                        <checksumPolicy>warn</checksumPolicy>
                    </snapshots>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>daily</updatePolicy>
                        <checksumPolicy>fail</checksumPolicy>
                    </releases>
                </pluginRepository>
                <pluginRepository>
                    <id>nm-internal</id>
                    <name>Numerical Method Repositories</name>
                    <url>http://repo.numericalmethod.com:8080/artifactory/internal/</url>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                        <checksumPolicy>warn</checksumPolicy>
                    </snapshots>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>daily</updatePolicy>
                        <checksumPolicy>fail</checksumPolicy>
                    </releases>
                </pluginRepository>
            </pluginRepositories>
        </profile>
        <!-- Uncomment the block below if you need to do deployment -->
        <!--
        <profile>
            <id>snapshot</id>
            <properties>
                <repositoryId>nm-repo</repositoryId>
                <url>http://repo.numericalmethod.com:8080/artifactory/snapshots/</url>
                <altDeploymentRepository>${repositoryId}::default::${url}</altDeploymentRepository>
            </properties>
        </profile>
        <profile>
            <id>release</id>
            <properties>
                <repositoryId>nm-repo</repositoryId>
                <url>http://repo.numericalmethod.com:8080/artifactory/releases/</url>
                <altDeploymentRepository>${repositoryId}::default::${url}</altDeploymentRepository>
            </properties>
        </profile>
        -->
    </profiles>
​
    <activeProfiles>
        <activeProfile>default</activeProfile>
    </activeProfiles>
​
    <!-- Put your deployment login below -->
    <servers>
    <!--
        <server>
            <id>nm-repo</id>
            <username>yourusername</username>
            <password>yourpassword</password>
        </server>
    -->
        <server>
            <id>nm-internal</id>
            <username>internal</username>
            <password>nm.internal.2021</password>
        </server>
	</servers>
​
    <interactiveMode>false</interactiveMode>
​
    <mirrors>
          <mirror>
               <id>maven-default-http-blocker</id>
               <mirrorOf>dummy</mirrorOf>
               <name>Dummy mirror to override default blocking mirror that blocks http</name>
               <url>http://0.0.0.0/</url>
         </mirror>
    </mirrors>

</settings>
```

### Note: Most of NM's repository are in HTTP protocol which is not supported thus we need to add mirrors to our original settings.xml

This problem can be further understood by the following answer (ref: StackOverflow)

<br/>

> More and more repositories use HTTPS nowadays, but this hasn’t always been the case. This means that Maven Central contains POMs with custom repositories that refer to a URL over HTTP. This makes downloads via such repository a target for a MITM attack. At the same time, developers are probably not aware that for some downloads an insecure URL is being used. Because uploaded POMs to Maven Central are immutable, a change for Maven was required. To solve this, we extended the mirror configuration with parameter, and we added a new external:http:* mirror selector (like existing external:*), meaning “any external URL using HTTP”. The decision was made to block such external HTTP repositories by default: this is done by providing a mirror in the conf/settings.xml blocking insecure HTTP external URLs.

<br/>

This must be added to the original `settings.xml`
```
<mirrors>
          <mirror>
               <id>maven-default-http-blocker</id>
               <mirrorOf>dummy</mirrorOf>
               <name>Dummy mirror to override default blocking mirror that blocks http</name>
               <url>http://0.0.0.0/</url>
         </mirror>
    </mirrors>
```

<br/>

## Setting up the workflow
---------------------

Now, that we have set up the local environment for the actions, we now must set-up github to facilitate CI/CD.

We first need to set-up a custom actions with an empty `.yaml` file.

Then paste the following code inside it.

```
name: Java CI with Maven

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'temurin'
          cache: maven


      - name: Build s2-app
        working-directory: ./s2-java/s2-app
        run: mvn -B package --file pom.xml

      - name: what is in the target
        working-directory: ./s2-java/s2-app/target
        run: ls -a

```
