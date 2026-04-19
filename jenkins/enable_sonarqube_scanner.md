
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
<img width="835" height="552" alt="image" src="https://github.com/user-attachments/assets/393f9442-7617-482e-b5b2-89d41467d5e8" />


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

<img width="943" height="552" alt="image" src="https://github.com/user-attachments/assets/7e0ee5ad-2587-46d8-b5a3-cf05ffc8e1f4" />


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

<img width="932" height="604" alt="image" src="https://github.com/user-attachments/assets/27a0436c-9202-47b4-97d1-9576e396abec" />


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

    environment {
      SONAR_TOKEN = credentials('sonarqube')
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

      // Optional but recommended
      // stage('Quality Gate') {
      //   steps {
      //     timeout(time: 5, unit: 'MINUTES') {
      //       waitForQualityGate abortPipeline: true
      //     }
      //   }
      // }

      stage('Show Sonar Issues') {
        steps {
          script {
            def response = sh(
              script: '''
                #curl -s -u $SONAR_TOKEN: \
                #"https://sonarcloud.io/api/issues/search?componentKeys=devsecaiops2_java-testing&pageSize=100"
                                  
                curl -s -u $SONAR_TOKEN: \\
                "https://sonarcloud.io/api/issues/search?componentKeys=devsecaiops2_java-testing&severities=CRITICAL,MAJOR&types=VULNERABILITY,BUG"
              ''',
              returnStdout: true
            ).trim()

            def json = readJSON text: response

            if (json.total > 0) {
              echo "🚨 Found ${json.total} issues:\n"

              json.issues.each { issue ->
                def line = issue.textRange?.startLine ?: "N/A"

                echo """
Type: ${issue.type}
Severity: ${issue.severity}
File: ${issue.component}
Line: ${line}
Message: ${issue.message}
----------------------------------------
"""
              }

              // Stop only if vulnerabilities exist
              def vulnCount = json.issues.findAll { it.type == 'VULNERABILITY' }.size()

              if (vulnCount > 0) {
                error("❌ Pipeline stopped due to ${vulnCount} vulnerabilities")
              }

            } else {
              echo "✅ No issues found"
            }
          }
        }
      }

      stage('Next Step') {
        steps {
          echo "🚀 Build is safe, continuing pipeline..."
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

<img width="887" height="404" alt="image" src="https://github.com/user-attachments/assets/aca526bf-f9f2-453d-a2fa-ea269af571be" />


---

# ✅ Expected Result

After successful run:

* Project appears in SonarCloud
* Issues detected:

  * Code smells
  * Security issues (e.g. hardcoded password)
* Dashboard populated

<img width="1761" height="809" alt="image" src="https://github.com/user-attachments/assets/fb782bff-7cce-4c05-904e-1cb91e135034" />

📊 Example Output
🚨 Found 2 issues:

Type: VULNERABILITY
Severity: CRITICAL
File: src/main/java/com/example/App.java
Line: 5
Message: Hardcoded password detected
----------------------------------------
---

<img width="835" height="643" alt="image" src="https://github.com/user-attachments/assets/b35fd94f-d4a1-4e76-bc00-1fc6d1baa01a" />

<img width="905" height="855" alt="image" src="https://github.com/user-attachments/assets/cebc3a44-2a59-4121-8054-eb24a2c07250" />


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

| Issue                        | Cause                | Fix                            |
| ---------------------------- | -------------------- | ------------------------------ |
| `sonar.organization missing` | Using SonarCloud     | Add organization               |
| `404 error`                  | Project not in org   | Create/import project          |
| Jenkins error                | Name mismatch        | Match `withSonarQubeEnv`       |
| No scan result               | Wrong projectKey     | Copy exact key                 |
| readJSON missing             | Plugin not installed | Install Pipeline Utility Steps |

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
