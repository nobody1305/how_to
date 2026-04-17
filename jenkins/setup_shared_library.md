# Jenkins Shared Library Setup Guide

This guide explains how to configure and use a Jenkins Shared Library from GitLab.

---

## 📦 1. Repository Structure (Required)

Your shared library repository **must follow this structure**:

```
jenkins-shared-library/
├── vars/
│   └── AWSPipeline.groovy
└── src/ (optional)
```

* `vars/` → contains global pipeline functions
* File name must match function name:

  * `AWSPipeline.groovy` → `AWSPipeline()`

---

## 🧾 2. Example Shared Library Function

`vars/AWSPipeline.groovy`:

```groovy
def call(Map pipelineParams) {

  pipeline {
    agent any

    stages {
      stage('Test') {
        steps {
          echo "TF Version: ${pipelineParams.TF_VERSION}"
        }
      }
    }

    post {
      success {
        echo 'SUCCESS'
      }
      failure {
        echo 'FAILED'
      }
    }
  }
}
```

---

## ⚙️ 3. Configure Global Pipeline Library in Jenkins

Go to:

```
Manage Jenkins → System → Global Pipeline Libraries
```
<img width="1700" height="618" alt="image" src="https://github.com/user-attachments/assets/ee3972d7-6e26-4fce-a649-e9a41e55875a" />

<img width="1591" height="726" alt="image" src="https://github.com/user-attachments/assets/22bba1bf-cfa4-49b3-864c-c31df11e703f" />


Add a new library:

* **Name**:

  ```
  jenkins-shared-library
  ```

* **Default version (branch)**:

  ```
  main
  ```

* **Retrieval method**:

  ```
  Modern SCM
  ```

* **SCM**:

  * Git
  * Repository URL:

    ```
    git@gitlab.com:<your-namespace>/jenkins-shared-library.git
    ```
  * Credentials: SSH key (git access)
 
* **Path**:
  you can leave this empty

Save the configuration.

---

## 🔐 4. SSH Setup (Required for GitLab)

<img width="595" height="671" alt="image" src="https://github.com/user-attachments/assets/2c636d7b-a05b-4902-8422-806652306360" />


1. Generate SSH key:

   ```
    ssh-keygen -t rsa -C "jenkins"
   ```

2. Add public key to GitLab (Settings → SSH Keys)

3. Add private key in Jenkins:

   ```
   Manage Jenkins → Credentials → Add Credentials
   ```

   * Type: SSH Username with private key
   * Username: `git`

4. Add GitLab to known_hosts inside Jenkins container:

   ```
   ssh-keyscan gitlab.com >> /var/jenkins_home/.ssh/known_hosts
   # or
   ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
   ```

---

## 🚀 5. Use Shared Library in Jenkinsfile

<img width="1656" height="745" alt="image" src="https://github.com/user-attachments/assets/dacaf9bc-26d9-4ee6-8529-9c0b7d78bbb3" />

<img width="1314" height="666" alt="image" src="https://github.com/user-attachments/assets/c3a29708-74a6-4997-a75b-1545c36dbc4e" />

### Basic Usage

Create a `Jenkinsfile` in your project:

```groovy
@Library('jenkins-shared-library@main') _
AWSPipeline(
  TF_VERSION: '1.5.1'
)
```

---

## ⚠️ Important Notes

* Always specify branch:

  ```
  @Library('jenkins-shared-library@main')
  ```
* Do NOT define another `pipeline {}` in Jenkinsfile if it already exists in shared library
* Ensure file is inside `vars/`, not root directory
* File name must match function name exactly (case-sensitive)

---

## 🧹 6. Troubleshooting

### Error: `No version specified`

→ Add `@main` or set default version in Jenkins

---

### Error: `expected to contain src or vars`

→ Ensure:

```
vars/AWSPipeline.groovy
```

exists in the correct branch

---

### Jenkins using old code

Clear cache:

```
rm -rf /var/jenkins_home/caches/*
docker restart jenkins
```

---

### SSH issues

Test inside container:

```
ssh -T git@gitlab.com
```
if this not working repeat step inside container or master jenkins

<img width="478" height="46" alt="image" src="https://github.com/user-attachments/assets/65d7bbc3-06e4-4d37-9fff-098ba8b0dc9a" />



---

## ✅ Result

After setup:

* Jenkins loads shared library from GitLab
* Pipeline can be reused across multiple jobs
* Centralized DevSecOps workflow is enabled

---
