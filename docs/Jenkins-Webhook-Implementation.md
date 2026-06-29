# Jenkins and Webhook Implementation Guide

## Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Jenkins Installation on macOS](#jenkins-installation-on-macos)
4. [Jenkins Initial Setup](#jenkins-initial-setup)
5. [GitHub Webhook Configuration](#github-webhook-configuration)
6. [Pipeline Configuration](#pipeline-configuration)
7. [Testing and Verification](#testing-and-verification)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

---

## Overview

**Jenkins** is an open-source automation server that helps automate parts of software development like building, testing, and deploying applications. **Webhooks** are user-defined HTTP callbacks triggered by specific events. When integrated, GitHub webhooks can automatically trigger Jenkins builds when you push code, create pull requests, or perform other actions.

### How They Work Together

1. You push code to GitHub
2. GitHub sends a webhook notification to Jenkins
3. Jenkins receives the webhook and automatically triggers a build
4. Jenkins checks out your code, installs dependencies, runs tests, and deploys
5. Build status is reported back to GitHub

---

## Prerequisites

### System Requirements
- **macOS**: 10.12 (Sierra) or later
- **RAM**: Minimum 2GB, recommended 4GB+
- **Disk Space**: At least 5GB free
- **Java**: Java 11 or later (required by Jenkins)

### Software to Install
- Java Development Kit (JDK)
- Git
- Node.js and npm
- Homebrew (macOS package manager)

### Access Requirements
- GitHub account with admin access to your repository
- macOS administrative access for installation
- SSH key pair (or use HTTPS with personal access tokens)

---

## Jenkins Installation on macOS

### Step 1: Install Java

Jenkins requires Java to run. Check if Java is already installed:

```bash
java -version
```

If not installed, install OpenJDK 11 or later using Homebrew:

```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Java
brew install openjdk@11

# Create a symbolic link (if needed)
sudo ln -sfn /usr/local/opt/openjdk@11/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-11.jdk
```

Verify installation:

```bash
java -version
```

### Step 2: Install Jenkins

Using Homebrew:

```bash
# Add Jenkins tap
brew tap jenkins-x/jx

# Install Jenkins
brew install jenkins-lts

# Start Jenkins service
brew services start jenkins-lts
```

### Step 3: Verify Jenkins Installation

Jenkins runs on `http://localhost:8080` by default. Open it in your browser:

```bash
open http://localhost:8080
```

You should see the Jenkins unlock page. Retrieve the initial admin password:

```bash
cat ~/.jenkins/secrets/initialAdminPassword
```

Copy this password and paste it into the browser.

---

## Jenkins Initial Setup

### Step 1: Unlock Jenkins

1. Paste the initial admin password from above
2. Click "Continue"

### Step 2: Install Suggested Plugins

1. Click "Install suggested plugins"
2. Jenkins will install essential plugins (this takes 5-10 minutes)

Plugins installed include:
- Pipeline
- Git
- GitHub Integration
- Email notifications

### Step 3: Create First Admin User

1. Fill in the form with your details:
   - **Username**: jenkins-admin
   - **Password**: (strong password)
   - **Full name**: Your Name
   - **Email**: your-email@example.com

2. Click "Save and Continue"

### Step 4: Configure Jenkins URL

1. Keep the default: `http://localhost:8080/`
2. Click "Save and Finish"
3. Click "Start using Jenkins"

### Step 5: Install Additional Plugins

1. Go to **Manage Jenkins** → **Plugin Manager**
2. Search for and install these plugins:
   - `Pipeline`
   - `GitHub`
   - `GitHub API`
   - `GitHub Branch Source`
   - `Git`
   - `NodeJS`
   - `Email Extension`

Installation steps:
1. Search for plugin name
2. Check the checkbox
3. Click "Install without restart"
4. Wait for installation to complete

### Step 6: Configure System Settings

1. Go to **Manage Jenkins** → **Configure System**

2. **Jenkins Location**:
   - Jenkins URL: `http://localhost:8080/`

3. **GitHub Server Configuration**:
   - Click "Add" next to "GitHub Servers"
   - Name: `GitHub`
   - API URL: `https://api.github.com`
   - Credentials: (set up in next section)

4. **Node.js Configuration**:
   - Go to **Manage Jenkins** → **Tools Configuration**
   - Look for "NodeJS installations"
   - Click "Add NodeJS"
   - Name: `Node 18`
   - Version: `18.x`
   - Check "Install automatically"

5. Save changes

---

## GitHub Webhook Configuration

### Step 1: Generate GitHub Personal Access Token

1. Go to GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. Click "Generate new token"
3. Give it a descriptive name: "Jenkins CI/CD"
4. Select scopes:
   - ✅ `repo` (Full control of private repositories)
   - ✅ `admin:repo_hook` (Write access to hooks in public and private repositories)
   - ✅ `admin:org_hook` (Full control of organization hooks) - if using org repos
5. Click "Generate token"
6. **Copy the token** (you won't see it again)

### Step 2: Add GitHub Credentials to Jenkins

1. In Jenkins, go to **Manage Jenkins** → **Manage Credentials**
2. Click on **"(global)"** under "Stores scoped to Jenkins"
3. Click "Add Credentials"
4. Fill in the form:
   - **Kind**: Username with password
   - **Scope**: Global
   - **Username**: (your GitHub username)
   - **Password**: (paste your personal access token)
   - **ID**: `github-credentials`
   - **Description**: GitHub API Token
5. Click "Create"

### Step 3: Create GitHub Webhook

1. Go to your GitHub repository
2. Navigate to **Settings** → **Webhooks**
3. Click "Add webhook"
4. Fill in the details:
   - **Payload URL**: `http://your-jenkins-url:8080/github-webhook/`
   - **Content type**: `application/json`
   - **Which events would you like to trigger this webhook?**: Select:
     - ✅ Push events
     - ✅ Pull requests
   - **Active**: ✅ (checked)
5. Click "Add webhook"

### Step 4: Verify Webhook Connection

1. In the webhook settings, scroll down to "Recent Deliveries"
2. Click on the most recent delivery
3. You should see a green checkmark (status 200)
4. If status is not 200, check troubleshooting section

---

## Pipeline Configuration

### Step 1: Create Jenkins Pipeline Job

1. In Jenkins home page, click "Create a job"
2. Enter job name: `amandevship-portfolio-ci`
3. Select "Pipeline"
4. Click "OK"

### Step 2: Configure Job Settings

1. **General**:
   - Check "GitHub project"
   - Project URL: `https://github.com/yourusername/amandevship-portfolio-react`

2. **Build Triggers**:
   - Check "GitHub hook trigger for GITscm polling"

3. **Pipeline**:
   - Definition: "Pipeline script from SCM"
   - SCM: Git
   - Repository URL: `https://github.com/yourusername/amandevship-portfolio-react.git`
   - Credentials: Select "github-credentials"
   - Branch: `*/main` (or your default branch)
   - Script Path: `Jenkinsfile`

4. Click "Save"

### Step 3: Create Jenkinsfile

Create a `Jenkinsfile` in your project root:

```groovy
pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }
    
    environment {
        NODE_ENV = 'production'
        DEPLOY_DIR = '/var/www/amandevship-portfolio'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing npm dependencies...'
                sh '''
                    cd client
                    npm install
                    cd ../server
                    npm install
                    cd ..
                '''
            }
        }
        
        stage('Build Client') {
            steps {
                echo 'Building React application...'
                sh '''
                    cd client
                    npm run build
                    cd ..
                '''
            }
        }
        
        stage('Build Server') {
            steps {
                echo 'Building TypeScript server...'
                sh '''
                    cd server
                    npm run build
                    cd ..
                '''
            }
        }
        
        stage('Run Tests') {
            steps {
                echo 'Running tests...'
                sh '''
                    cd client
                    npm run test -- --passWithNoTests
                    cd ..
                '''
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                echo 'Deploying to staging environment...'
                sh '''
                    # Deploy logic here
                    echo "Deploying to staging..."
                '''
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying to production environment...'
                sh '''
                    # Deploy logic here
                    echo "Deploying to production..."
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Build completed!'
            // Clean up
            cleanWs()
        }
        success {
            echo 'Build was successful!'
            // Send success notification
        }
        failure {
            echo 'Build failed!'
            // Send failure notification
        }
    }
}
```

Commit this file to your repository:

```bash
git add Jenkinsfile
git commit -m "Add Jenkins pipeline configuration"
git push origin main
```

---

## Testing and Verification

### Step 1: Manual Build Trigger

1. In Jenkins, go to your pipeline job
2. Click "Build Now"
3. Click on the build number in the "Build History"
4. View the console output to verify:
   - Code checkout ✅
   - Dependency installation ✅
   - Build completion ✅
   - Tests passing ✅

### Step 2: Test Webhook Trigger

1. Make a small change to your code locally:

```bash
# Make a change
echo "# Test trigger" >> README.md

# Commit and push
git add README.md
git commit -m "Test webhook trigger"
git push origin main
```

2. Within seconds, Jenkins should automatically start a build
3. Go to Jenkins and verify the build appears in the build history

### Step 3: Monitor Build Logs

During a build, click on the build number and view "Console Output" to see:
- Checkout progress
- Dependency installation
- Build output
- Test results
- Deployment status

### Step 4: Verify Build Status in GitHub

1. Go to your GitHub repository
2. In your recent commit, you should see a build status indicator
3. Click on it to see Jenkins build details

---

## Troubleshooting

### Issue 1: Webhook Not Triggering Builds

**Symptoms**: Push to GitHub but no build starts in Jenkins

**Solutions**:
1. Check webhook delivery in GitHub:
   - Go to repo **Settings** → **Webhooks**
   - Click on webhook
   - Check "Recent Deliveries" tab
   - Look for status codes (should be 200 or 202)

2. If delivery failed, check Jenkins logs:
   ```bash
   # View Jenkins logs on macOS
   brew services log jenkins-lts
   ```

3. Verify GitHub credentials in Jenkins:
   - **Manage Jenkins** → **Credentials** → verify token is correct

4. Check job configuration:
   - Job **Configure** → **Build Triggers** → verify "GitHub hook trigger" is enabled

### Issue 2: Build Fails Due to Missing Dependencies

**Symptoms**: Build fails at "Install Dependencies" stage

**Solutions**:
1. SSH into build machine and test manually:
   ```bash
   cd /tmp
   git clone https://github.com/yourusername/amandevship-portfolio-react.git
   cd amandevship-portfolio-react/client
   npm install
   ```

2. Check Node.js version in Jenkins:
   - **Manage Jenkins** → **Tools Configuration** → NodeJS installations
   - Verify Node 18+ is configured

3. Check network connectivity for npm registry:
   ```bash
   npm config get registry
   ```

### Issue 3: SSH Key Authentication Failures

**Symptoms**: "Permission denied (publickey)" in build logs

**Solutions**:
1. Use HTTPS instead of SSH in repository URL
2. Or set up SSH keys:
   ```bash
   # Generate SSH key
   ssh-keygen -t ed25519 -C "jenkins@localhost" -f ~/.ssh/jenkins_key
   
   # Add to GitHub
   # Go to GitHub → Settings → SSH keys
   # Add ~/.ssh/jenkins_key.pub content
   
   # In Jenkins, add SSH credentials
   # Manage Jenkins → Credentials → Add SSH key
   ```

### Issue 4: Build Takes Too Long / Timeout

**Symptoms**: Build stops after 30 minutes with timeout error

**Solutions**:
1. Increase timeout in Jenkinsfile:
   ```groovy
   options {
       timeout(time: 60, unit: 'MINUTES')
   }
   ```

2. Optimize npm install:
   ```bash
   npm ci  # Use instead of npm install in CI/CD
   ```

3. Cache dependencies:
   ```bash
   npm install --prefer-offline --no-audit
   ```

### Issue 5: Deployment Fails

**Symptoms**: Build succeeds but deployment stage fails

**Solutions**:
1. Check deployment credentials:
   - Verify SSH keys or deploy tokens are configured
   
2. Test deployment manually:
   ```bash
   # Test SSH connection to server
   ssh -i ~/.ssh/deploy_key user@deployment-server.com "echo 'Connection successful'"
   ```

3. Check deploy directory permissions:
   ```bash
   # On deployment server
   ls -la /var/www/amandevship-portfolio
   sudo chown -R jenkins:jenkins /var/www/amandevship-portfolio
   ```

### Issue 6: GitHub API Rate Limit

**Symptoms**: Webhook deliveries show 403 errors, build can't fetch GitHub data

**Solutions**:
1. In Jenkins, verify you're using a Personal Access Token (not username/password)
2. GitHub increases rate limits for authenticated requests
3. Check current rate limit:
   ```bash
   curl -u username:token https://api.github.com/rate_limit
   ```

### Issue 7: Jenkins Won't Start

**Symptoms**: Jenkins service fails to start on macOS

**Solutions**:
1. Check Java installation:
   ```bash
   java -version
   ```

2. View detailed error logs:
   ```bash
   cat /usr/local/var/log/jenkins.log
   ```

3. Restart the service:
   ```bash
   brew services restart jenkins-lts
   ```

### Issue 8: Port 8080 Already in Use

**Symptoms**: Jenkins fails to start because port 8080 is already in use

**Solutions**:
1. Find process using port 8080:
   ```bash
   lsof -i :8080
   ```

2. Kill the process:
   ```bash
   kill -9 <PID>
   ```

3. Or change Jenkins port:
   ```bash
   # Edit Jenkins config
   nano /usr/local/etc/rc.d/jenkins
   
   # Change: --httpPort=8080
   # To: --httpPort=8081
   
   # Restart
   brew services restart jenkins-lts
   ```

---

## Best Practices

### 1. Security

- **Secure Jenkins**:
  ```bash
  # Go to Manage Jenkins → Configure Global Security
  # Enable "Enable security" checkbox
  # Use GitHub OAuth for user authentication
  ```

- **Credentials Management**:
  - Use Jenkins Credentials Store, not environment variables
  - Rotate access tokens regularly
  - Use different tokens for different services

- **Firewall Rules**:
  ```bash
  # Only allow GitHub webhook IPs
  # https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-githubs-ip-addresses
  ```

### 2. Pipeline Best Practices

- **Always use credentials via Jenkins**:
  ```groovy
  // Bad
  sh "npm publish --token=abc123"
  
  // Good
  withCredentials([string(credentialsId: 'npm-token', variable: 'NPM_TOKEN')]) {
      sh "npm publish --token=$NPM_TOKEN"
  }
  ```

- **Implement Retry Logic**:
  ```groovy
  stage('Install Dependencies') {
      steps {
          retry(3) {
              sh 'npm install'
          }
      }
  }
  ```

- **Use Timeout**:
  ```groovy
  stage('Build') {
      steps {
          timeout(time: 15, unit: 'MINUTES') {
              sh 'npm run build'
          }
      }
  }
  ```

### 3. Notifications

- **Email Notifications**:
  ```groovy
  post {
      failure {
          mail to: 'dev@example.com',
               subject: "Build Failed: ${env.JOB_NAME}",
               body: "Build failed. Check Jenkins: ${env.BUILD_URL}"
      }
  }
  ```

- **Slack Notifications**:
  1. Install "Slack Notification" plugin
  2. Configure webhook in Slack workspace
  3. Add to Jenkinsfile:
     ```groovy
     post {
         failure {
             slackSend(color: 'danger',
                      message: "Build Failed: ${env.JOB_NAME}")
         }
     }
     ```

### 4. Build Artifacts

- **Archive Build Outputs**:
  ```groovy
  post {
      always {
          archiveArtifacts artifacts: 'client/dist/**/*',
                           allowEmptyArchive: true
      }
  }
  ```

### 5. Version Control for Jenkins Configuration

- **Export Job Configuration**:
  ```bash
  # Backup Jenkins jobs
  tar -czf jenkins-jobs-backup.tar.gz ~/.jenkins/jobs/
  
  # Restore
  tar -xzf jenkins-jobs-backup.tar.gz -C ~/.jenkins/
  ```

---

## Step-by-Step Deployment Example

### Deploy to Remote Server via SSH

1. **Generate SSH Key**:
   ```bash
   ssh-keygen -t ed25519 -C "jenkins-deploy" -f ~/.ssh/deploy_key
   ```

2. **Add to Jenkins Credentials**:
   - **Manage Jenkins** → **Credentials** → **Add SSH Credentials**
   - Load private key from `~/.ssh/deploy_key`
   - ID: `deploy-ssh-key`

3. **Add Deployment Stage to Jenkinsfile**:
   ```groovy
   stage('Deploy to Production') {
       when {
           branch 'main'
       }
       steps {
           sshagent(['deploy-ssh-key']) {
               sh '''
                   ssh -o StrictHostKeyChecking=no deploy@production.com << 'EOF'
                   cd /var/www/amandevship-portfolio
                   git pull origin main
                   npm install
                   npm run build
                   pm2 restart app
                   EOF
               '''
           }
       }
   }
   ```

---

## Monitoring and Maintenance

### Enable Build Logs Archival

```groovy
post {
    always {
        archiveArtifacts artifacts: 'client/dist/**/*',
                         allowEmptyArchive: true
        
        // Delete old builds after 30 days
        logRotator(daysToKeepStr: '30',
                   numToKeepStr: '10',
                   artifactDaysToKeepStr: '5',
                   artifactNumToKeepStr: '5')
    }
}
```

### Monitor Webhook Health

Create a simple script to monitor webhook deliveries:

```bash
#!/bin/bash
# monitor-webhooks.sh

TOKEN="your-github-token"
REPO="yourusername/amandevship-portfolio-react"

curl -s -H "Authorization: token $TOKEN" \
     "https://api.github.com/repos/$REPO/hooks" | \
     jq '.[].events, .[].last_response'
```

Run periodically:
```bash
chmod +x monitor-webhooks.sh
./monitor-webhooks.sh
```

---

## Quick Reference Commands

```bash
# Start Jenkins
brew services start jenkins-lts

# Stop Jenkins
brew services stop jenkins-lts

# Restart Jenkins
brew services restart jenkins-lts

# View Jenkins logs
brew services log jenkins-lts

# Jenkins running on
open http://localhost:8080

# Update Jenkins
brew upgrade jenkins-lts

# Uninstall Jenkins
brew uninstall jenkins-lts
rm -rf ~/.jenkins
```

---

## Resources and References

- **Official Jenkins Documentation**: https://jenkins.io/doc/
- **Jenkins Plugins**: https://plugins.jenkins.io/
- **GitHub Webhook Documentation**: https://docs.github.com/en/developers/webhooks-and-events/webhooks
- **GitHub API Documentation**: https://docs.github.com/en/rest
- **Jenkins on macOS**: https://www.jenkins.io/download/thank-you/macos

---

## Support and Next Steps

### Next Steps After Setup

1. ✅ Install Jenkins on your Mac
2. ✅ Configure GitHub credentials
3. ✅ Create webhook in GitHub
4. ✅ Set up Jenkins pipeline job
5. ✅ Commit Jenkinsfile to repository
6. ✅ Make a test push to verify webhook trigger
7. ✅ Monitor build in Jenkins
8. ✅ Configure notifications
9. ✅ Set up deployment stage
10. ✅ Enable build artifact archival

### Getting Help

If you encounter issues:

1. **Check Jenkins logs**: `brew services log jenkins-lts`
2. **Check webhook deliveries** in GitHub settings
3. **Verify credentials** in Jenkins Credentials Manager
4. **Test components individually** (Git, Node.js, npm)
5. **Check Jenkins community**: https://www.jenkins.io/participate/

---

**Created**: May 2026  
**For**: amandevship-portfolio-react  
**macOS Implementation Guide**
