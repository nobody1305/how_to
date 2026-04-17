# Jenkins Shared Library Setup Guide

Complete guide to setup and use Jenkins Shared Library with GitLab integration.

---

## Table of Contents
1. [Repository Structure](#repository-structure)
2. [Create Shared Library](#create-shared-library)
3. [Setup SSH Authentication](#setup-ssh-authentication)
4. [Configure Jenkins](#configure-jenkins)
5. [Usage in Pipeline](#usage-in-pipeline)
6. [Troubleshooting](#troubleshooting)

---

## Repository Structure

Your Jenkins Shared Library repository must have this structure:

```
jenkins-shared-library/
├── vars/
│   ├── AWSPipeline.groovy
│   └── myFunction.groovy
├── src/
│   └── com/
│       └── company/
│           └── Utils.groovy
└── resources/
    └── scripts/
        └── deploy.sh
```

**Minimum requirement**: At least one of `vars/` or `src/` directories must exist.

---

## Create Shared Library

### 1. Create Repository Structure

```bash
cd /path/to/your/jenkins-shared-library
mkdir -p vars
```

### 2. Create Pipeline Function

Create `vars/AWSPipeline.groovy`:

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

### 3. Commit and Push

```bash
git add vars/
git commit -m "Add Jenkins shared library structure"
git push origin main
```

---

## Setup SSH Authentication

### For Jenkins Running in Docker

#### 1. Access Jenkins Container

```bash
docker exec -it <container-id> bash
```

#### 2. Generate SSH Key

```bash
cd ~
mkdir -p .ssh
chmod 700 .ssh
ssh-keygen -t rsa -b 4096 -C "jenkins@docker"
# Press Enter 3 times (default location, no passphrase)
```

#### 3. Copy Public Key

```bash
cat ~/.ssh/id_rsa.pub
```

#### 4. Add to GitLab

- Go to **GitLab → Settings → SSH Keys**
- Paste the public key
- Click **Add key**

#### 5. Add GitLab to Known Hosts

```bash
ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
```

#### 6. Test SSH Connection

```bash
ssh -T git@gitlab.com
```

Expected output: `Welcome to GitLab, @username!`

#### 7. Copy Private Key

```bash
cat ~/.ssh/id_rsa
```

Keep this output for the next step.

---

## Configure Jenkins

### 1. Add SSH Credentials

- Go to **Manage Jenkins → Manage Credentials**
- Click **System → Global credentials → Add Credentials**
- Configure:
  - **Kind**: `SSH Username with private key`
  - **ID**: `gitlab-ssh-key`
  - **Username**: `git`
  - **Private Key**: Select "Enter directly" and paste the private key from previous step
- Click **Create**

### 2. Configure Global Pipeline Library

- Go to **Manage Jenkins → Configure System**
- Scroll to **Global Pipeline Libraries**
- Click **Add**
- Configure:

```
Name: jenkins-shared-library
Default version: main
☐ Load implicitly
☑ Allow default version to be overridden
☑ Include @Library changes in job recent changes

Retrieval method: Modern SCM

Source Code Management: Git
  Project Repository: git@gitlab.com:your-org/jenkins-shared-library.git
  Credentials: gitlab-ssh-key
  
Behaviors:
  ☑ Discover branches
  ☐ Fresh clone per build (leave unchecked for better performance)
```

- Click **Save**

---

## Usage in Pipeline

### Basic Usage

Create a `Jenkinsfile` in your project:

```groovy
@Library('jenkins-shared-library@main') _

AWSPipeline(
  TF_VERSION: '1.5.1'
)
```

### Load Specific Version

```groovy
@Library('jenkins-shared-library@v1.2.3') _
```

### Load Specific Commit

```groovy
@Library('jenkins-shared-library@3bf9b58') _
```

### Load Multiple Libraries

```groovy
@Library(['lib1@master', 'lib2@v1.0']) _
```

### Dynamic Loading

```groovy
library 'jenkins-shared-library@main'
```

---

## Troubleshooting

### Error: "Library expected to contain at least one of src or vars directories"

**Cause**: Repository doesn't have `vars/` or `src/` directory.

**Solution**:
```bash
cd jenkins-shared-library
mkdir -p vars
# Add your .groovy files
git add vars/
git commit -m "Add vars directory"
git push origin main
```

### Error: "Permission denied (publickey)"

**Cause**: SSH key not configured or not added to GitLab.

**Solution**:
1. Verify SSH connection: `ssh -T git@gitlab.com`
2. Regenerate SSH key if needed
3. Ensure public key is added to GitLab
4. Ensure private key is added to Jenkins credentials

### Error: Library not loading latest changes

**Solution**:
1. Check you're using the correct branch: `@Library('jenkins-shared-library@main')`
2. Verify changes are pushed to GitLab
3. Try enabling "Fresh clone per build" in Jenkins library configuration
4. Clear Jenkins workspace and rebuild

### Error: Function not found

**Cause**: Function name doesn't match filename.

**Solution**:
- File: `vars/AWSPipeline.groovy` → Call: `AWSPipeline()`
- File: `vars/myFunction.groovy` → Call: `myFunction()`
- Filenames are case-sensitive

---

## Alternative: HTTPS with Token

If SSH is problematic, use HTTPS with Personal Access Token:

### 1. Create GitLab Token

- GitLab → **Settings → Access Tokens**
- Name: `jenkins-library`
- Scopes: `read_repository`
- Create and copy token

### 2. Add to Jenkins

- **Manage Jenkins → Manage Credentials → Add**
- **Kind**: `Username with password`
- **Username**: Your GitLab username
- **Password**: Your personal access token
- **ID**: `gitlab-token`

### 3. Configure Library

```
Project Repository: https://gitlab.com/your-org/jenkins-shared-library.git
Credentials: gitlab-token
```

---

## Best Practices

1. **Version your library**: Use tags for stable releases
2. **Test changes**: Use feature branches before merging to main
3. **Document functions**: Add comments in your .groovy files
4. **Keep it simple**: Start with basic functions, add complexity as needed
5. **Use parameters**: Make functions reusable with Map parameters
6. **Error handling**: Add try-catch blocks in your functions

---

## Example: Advanced Pipeline Function

```groovy
def call(Map config = [:]) {
  // Set defaults
  def tfVersion = config.TF_VERSION ?: '1.5.0'
  def awsRegion = config.AWS_REGION ?: 'us-east-1'
  def environment = config.ENVIRONMENT ?: 'dev'
  
  pipeline {
    agent any
    
    environment {
      TF_VERSION = "${tfVersion}"
      AWS_REGION = "${awsRegion}"
      ENV = "${environment}"
    }
    
    stages {
      stage('Validate') {
        steps {
          echo "Validating Terraform ${tfVersion} in ${awsRegion}"
          sh 'terraform --version'
        }
      }
      
      stage('Plan') {
        steps {
          sh 'terraform plan'
        }
      }
      
      stage('Apply') {
        when {
          branch 'main'
        }
        steps {
          sh 'terraform apply -auto-approve'
        }
      }
    }
    
    post {
      success {
        echo "Deployment to ${environment} successful!"
      }
      failure {
        echo "Deployment failed!"
      }
    }
  }
}
```

Usage:
```groovy
@Library('jenkins-shared-library@main') _

AWSPipeline(
  TF_VERSION: '1.5.1',
  AWS_REGION: 'us-west-2',
  ENVIRONMENT: 'production'
)
```

---

## References

- [Jenkins Shared Libraries Documentation](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [GitLab SSH Keys](https://docs.gitlab.com/ee/user/ssh.html)
- [Jenkins Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
