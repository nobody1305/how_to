
# Jenkins + SonarCloud Integration (Java) — Complete Setup Guide

## 📌 Overview

This guide walks through the **end-to-end setup** of integrating a Java project with SonarCloud using Jenkins and a shared library.

---

# 🧱 Architecture

```text
Jenkins → Maven Build → SonarCloud Analysis
```

---

# ⚙️ STEP 1 — Setup SonarCloud

Using SonarCloud

## 1. Create Organization

* Go to SonarCloud
* Create or use existing org (example: `devsecaiops2`)

---

## 2. Create / Import Project

* Click **“Analyze new project”**
* Choose:

  * Import from GitLab
    OR
  * Manual setup
* Note:

  * `organization`
  * `projectKey`

Example:

```text
Organization: devsecaiops2
ProjectKey: devsecaiops2_java-testing
```

---

## 3. Generate Token

* Go to: **My Account → Security**
* Generate token
* Save it (used in Jenkins)

<img width="1150" height="807" alt="image" src="https://github.com/user-attachments/assets/bb5a6b6b-2673-426f-8d35-4ff6bd65dbd6" />

---

# ⚙️ STEP 2 — Setup Jenkins

Using Jenkins

---

## 1. Install Plugin

Go to:

```
Manage Jenkins → Plugins
```

Install:

* SonarQube Scanner for Jenkins

---

## 2. Configure Sonar Server

Go to:

```
Manage Jenkins → Configure System
```

Add SonarQube server:

* Name: `sonarqube-scanner`
* Server URL:

  ```
  https://sonarcloud.io
  ```
* Credentials:

  * Add token from SonarCloud

---

## 3. Configure Tools

Go to:

```
Manage Jenkins → Global Tool Configuration
```

Add:

### JDK

* Name: `JDK11`

### Maven

* Name: `Maven`

---

# ⚙️ STEP 3 — Prepare Java Project

## Structure

```text
simple-java-app/
├── pom.xml
└── src/main/java/com/example/App.java
```

---

## Example `App.java`

```java
package com.example;

public class App {
    public static void main(String[] args) {
        String password = "admin123"; // Sonar will flag this
        System.out.println(password);
    }
}
```

---

# ⚙️ STEP 4 — Jenkins Shared Library

## Jenkinsfile

```groovy
@Library('jenkins-shared-library@main') _
SonarPipeline(
  sonarServer: 'sonarqube-server',
  projectKey: 'test-java'
)
```

---

## Shared Library (`vars/SonarPipeline.groovy`)

```groovy
def call(Map config) {
  pipeline {
    agent any

    tools {
      jdk 'JDK11'
      maven 'Maven'
    }

    stages {
      stage('Build') {
        steps {
          sh 'mvn clean compile'
        }
      }

      stage('Sonar Scan') {
        steps {
          withSonarQubeEnv('sonarqube-scanner') {
            sh '''
              mvn sonar:sonar \
                -Dsonar.projectKey=devsecaiops2_java-testing \
                -Dsonar.organization=devsecaiops2 \
                -Dsonar.sources=src/main/java
            '''
          }
        }
      }
    }
  }
}
```

---

# ⚙️ STEP 5 — Run Pipeline

1. Create Jenkins job
2. Point to your repo
3. Run build

---

# ✅ Expected Result

After successful run:

* Project appears in SonarCloud
* Issues detected:

  * Code smells
  * Security issues (e.g. hardcoded password)
* Dashboard populated

---

# 🔍 Verification

Go to SonarCloud:

```
Organization → Project → Dashboard
```

Check:

* Bugs
* Vulnerabilities
* Code smells

---

# ⚠️ Common Issues & Fixes

| Issue                        | Cause              | Fix                      |
| ---------------------------- | ------------------ | ------------------------ |
| `sonar.organization missing` | Using SonarCloud   | Add organization         |
| `404 error`                  | Project not in org | Create/import project    |
| Jenkins error                | Name mismatch      | Match `withSonarQubeEnv` |
| No scan result               | Wrong projectKey   | Copy exact key           |

---

# 🧠 Best Practices

* Use **one organization**
* Naming convention:

  ```
  <org>_<repo-name>
  ```
* Keep shared library reusable
* Avoid hardcoding values (later improvement)

---

# 🚀 Next Improvements

* Add Quality Gate:

  ```groovy
  waitForQualityGate abortPipeline: true
  ```
* Integrate with GitLab MR
* Add dependency scanning (Snyk / Trivy)
* Automate onboarding

---

# 🎯 Final Outcome

You now have:

* ✅ Jenkins pipeline working
* ✅ SonarCloud integration complete
* ✅ Automated Java code scanning
* ✅ Reusable shared library

---

**End of Setup Guide**
