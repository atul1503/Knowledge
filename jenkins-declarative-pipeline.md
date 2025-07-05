# Jenkins Declarative Pipeline

## What is Jenkins Declarative Pipeline?

Jenkins Declarative Pipeline is a way to write instructions for Jenkins automation server using a structured format. It tells Jenkins what steps to follow when building, testing, or deploying your code. Think of it as a recipe that Jenkins follows automatically.

## Basic Structure

Every Jenkins declarative pipeline starts with the word `pipeline` and contains different sections called "blocks". Here is the most basic structure:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building the application'
            }
        }
    }
}
```

Let me explain each part:

- `pipeline`: This keyword tells Jenkins this is a declarative pipeline
- `agent any`: This tells Jenkins to run this pipeline on any available computer (called "agent")
- `stages`: This is a container that holds all the different stages of your pipeline
- `stage('Build')`: This creates a stage with the name "Build"
- `steps`: This contains the actual commands to run in this stage
- `echo 'Building the application'`: This prints a message to the console

## Agent Block

The `agent` block tells Jenkins where to run your pipeline. Here are different ways to specify agents:

### Run on Any Available Agent

```groovy
pipeline {
    agent any
    // ... rest of pipeline
}
```

- `agent any`: Jenkins will pick any available computer to run this pipeline

### Run on Specific Agent with Label

```groovy
pipeline {
    agent {
        label 'linux'
    }
    // ... rest of pipeline
}
```

- `label 'linux'`: This runs the pipeline only on computers that have the label "linux"

### Run on Agent with Docker

```groovy
pipeline {
    agent {
        docker {
            image 'node:14'
            args '-v /tmp:/tmp'
        }
    }
    // ... rest of pipeline
}
```

- `docker`: This tells Jenkins to run the pipeline inside a Docker container
- `image 'node:14'`: This specifies which Docker image to use (Node.js version 14)
- `args '-v /tmp:/tmp'`: These are extra arguments passed to Docker command

### No Global Agent

```groovy
pipeline {
    agent none
    stages {
        stage('Build') {
            agent any
            steps {
                echo 'Building'
            }
        }
    }
}
```

- `agent none`: This means no global agent is defined
- Each stage must then define its own agent

## Stages and Steps

Stages organize your pipeline into logical sections. Each stage contains steps that do the actual work.

### Multiple Stages Example

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                echo 'Getting code from repository'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Building the application'
                sh 'make build'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests'
                sh 'make test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying application'
                sh 'make deploy'
            }
        }
    }
}
```

- `checkout scm`: This downloads the code from your source control (like Git)
- `sh 'make build'`: This runs a shell command called "make build"
- Each stage runs one after another in order

## Environment Variables

You can define variables that can be used throughout your pipeline.

### Global Environment Variables

```groovy
pipeline {
    agent any
    environment {
        APP_NAME = 'my-application'
        VERSION = '1.0.0'
        DEPLOY_ENV = 'production'
    }
    stages {
        stage('Build') {
            steps {
                echo "Building ${APP_NAME} version ${VERSION}"
                sh "echo 'App: ${APP_NAME}, Version: ${VERSION}'"
            }
        }
    }
}
```

- `environment`: This block defines variables for the entire pipeline
- `APP_NAME = 'my-application'`: This creates a variable named APP_NAME
- `${APP_NAME}`: This is how you use the variable in your pipeline

### Stage-Specific Environment Variables

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            environment {
                BUILD_NUMBER = '123'
            }
            steps {
                echo "Build number is ${BUILD_NUMBER}"
            }
        }
    }
}
```

- Environment variables defined inside a stage are only available in that stage

## Parameters

Parameters allow users to provide input when running the pipeline.

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Environment to deploy')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip running tests')
    }
    stages {
        stage('Build') {
            steps {
                echo "Building branch: ${params.BRANCH_NAME}"
                echo "Target environment: ${params.ENVIRONMENT}"
                echo "Skip tests: ${params.SKIP_TESTS}"
            }
        }
    }
}
```

- `parameters`: This block defines input parameters for the pipeline
- `string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build')`: This creates a text input parameter
- `choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Environment to deploy')`: This creates a dropdown list
- `booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip running tests')`: This creates a checkbox
- `${params.BRANCH_NAME}`: This is how you use parameter values in your pipeline

## Triggers

Triggers tell Jenkins when to automatically run your pipeline.

```groovy
pipeline {
    agent any
    triggers {
        cron('H 2 * * *')
        pollSCM('H/5 * * * *')
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building application'
            }
        }
    }
}
```

- `triggers`: This block defines when to automatically run the pipeline
- `cron('H 2 * * *')`: This runs the pipeline every day at 2 AM
- `pollSCM('H/5 * * * *')`: This checks for code changes every 5 minutes and runs pipeline if changes found

## Options

Options control how the pipeline behaves.

```groovy
pipeline {
    agent any
    options {
        timeout(time: 1, unit: 'HOURS')
        retry(3)
        skipDefaultCheckout()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building application'
            }
        }
    }
}
```

- `options`: This block defines pipeline behavior settings
- `timeout(time: 1, unit: 'HOURS')`: This stops the pipeline if it runs longer than 1 hour
- `retry(3)`: This retries the pipeline up to 3 times if it fails
- `skipDefaultCheckout()`: This prevents automatic code checkout
- `buildDiscarder(logRotator(numToKeepStr: '10'))`: This keeps only the last 10 build logs

## Tools

Tools block automatically installs and configures tools needed for your pipeline.

```groovy
pipeline {
    agent any
    tools {
        maven 'Maven-3.8.1'
        jdk 'OpenJDK-11'
        nodejs 'NodeJS-14'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'node --version'
            }
        }
    }
}
```

- `tools`: This block defines which tools to install
- `maven 'Maven-3.8.1'`: This installs Maven version 3.8.1
- `jdk 'OpenJDK-11'`: This installs OpenJDK version 11
- `nodejs 'NodeJS-14'`: This installs Node.js version 14

## When Conditions

When conditions control whether a stage should run based on certain criteria.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building application'
            }
        }
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                echo 'Deploying to staging'
            }
        }
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying to production'
            }
        }
        stage('Run Integration Tests') {
            when {
                not {
                    branch 'main'
                }
            }
            steps {
                echo 'Running integration tests'
            }
        }
    }
}
```

- `when`: This block defines conditions for running a stage
- `branch 'develop'`: This stage only runs when building the 'develop' branch
- `not { branch 'main' }`: This stage runs when NOT building the 'main' branch

### More When Conditions

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                echo 'Deploying application'
            }
        }
        stage('Notify') {
            when {
                allOf {
                    branch 'main'
                    environment name: 'DEPLOY_ENV', value: 'production'
                }
            }
            steps {
                echo 'Sending notification'
            }
        }
    }
}
```

- `anyOf`: This runs the stage if ANY of the conditions are true
- `allOf`: This runs the stage if ALL of the conditions are true
- `environment name: 'DEPLOY_ENV', value: 'production'`: This checks if environment variable equals specific value

## Parallel Execution

You can run multiple stages at the same time to speed up your pipeline.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building application'
            }
        }
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo 'Running unit tests'
                        sh 'npm test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo 'Running integration tests'
                        sh 'npm run test:integration'
                    }
                }
                stage('Security Tests') {
                    steps {
                        echo 'Running security tests'
                        sh 'npm run test:security'
                    }
                }
            }
        }
    }
}
```

- `parallel`: This keyword makes the enclosed stages run at the same time
- All three test stages will run simultaneously instead of one after another

## Post Actions

Post actions run after the pipeline completes, regardless of success or failure.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building application'
            }
        }
    }
    post {
        always {
            echo 'This runs whether pipeline succeeds or fails'
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            mail to: 'team@company.com',
                 subject: 'Build Success',
                 body: 'The build was successful'
        }
        failure {
            echo 'Pipeline failed!'
            mail to: 'team@company.com',
                 subject: 'Build Failed',
                 body: 'The build failed'
        }
        unstable {
            echo 'Pipeline is unstable'
        }
        changed {
            echo 'Pipeline result changed from previous run'
        }
    }
}
```

- `post`: This block defines actions to run after pipeline completion
- `always`: This runs regardless of pipeline result
- `success`: This runs only if pipeline succeeds
- `failure`: This runs only if pipeline fails
- `unstable`: This runs if pipeline is unstable (tests failed but build succeeded)
- `changed`: This runs if pipeline result changed from previous run
- `cleanWs()`: This cleans up the workspace after pipeline completes

## Input Steps

Input steps pause the pipeline and wait for user approval.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building application'
            }
        }
        stage('Deploy') {
            steps {
                input message: 'Deploy to production?',
                      ok: 'Deploy',
                      submitter: 'admin,manager'
                echo 'Deploying to production'
            }
        }
    }
}
```

- `input`: This pauses the pipeline and waits for user input
- `message`: This is the message shown to the user
- `ok`: This is the text on the approval button
- `submitter`: This limits who can approve (comma-separated list)

## Script Block

Sometimes you need to write custom logic that doesn't fit declarative syntax. Use script blocks for this.

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def buildNumber = env.BUILD_NUMBER
                    def branchName = env.BRANCH_NAME
                    
                    if (branchName == 'main') {
                        echo "Building main branch, build number: ${buildNumber}"
                    } else {
                        echo "Building feature branch: ${branchName}"
                    }
                }
            }
        }
    }
}
```

- `script`: This allows you to write custom Groovy code
- `def buildNumber = env.BUILD_NUMBER`: This creates a variable with the build number
- `if (branchName == 'main')`: This is a conditional statement in Groovy

## Common Commands and Functions

### Shell Commands

```groovy
steps {
    sh 'echo "Hello World"'
    sh 'ls -la'
    sh 'npm install'
    sh 'docker build -t myapp .'
}
```

- `sh`: This runs shell commands on Unix/Linux systems

### Windows Commands

```groovy
steps {
    bat 'echo "Hello World"'
    bat 'dir'
    bat 'npm install'
}
```

- `bat`: This runs batch commands on Windows systems

### Archive Artifacts

```groovy
steps {
    archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
}
```

- `archiveArtifacts`: This saves files from the build for later use
- `artifacts: 'dist/**/*'`: This specifies which files to save
- `fingerprint: true`: This creates fingerprints for tracking file changes

### Publish Test Results

```groovy
steps {
    publishTestResults testResultsPattern: 'test-results.xml'
}
```

- `publishTestResults`: This publishes test results to Jenkins
- `testResultsPattern`: This specifies the pattern for test result files

## Complete Example

Here's a comprehensive example that combines many concepts:

```groovy
pipeline {
    agent any
    
    environment {
        APP_NAME = 'my-web-app'
        DOCKER_REGISTRY = 'my-registry.com'
        DEPLOY_ENV = 'staging'
    }
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['staging', 'production'], description: 'Environment to deploy')
        booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip running tests')
    }
    
    tools {
        nodejs 'NodeJS-14'
        dockerTool 'Docker-20'
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies'
                sh 'npm install'
            }
        }
        
        stage('Test') {
            when {
                not {
                    params.SKIP_TESTS
                }
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo 'Running unit tests'
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo 'Running integration tests'
                        sh 'npm run test:integration'
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building application'
                sh 'npm run build'
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
                    def imageName = "${DOCKER_REGISTRY}/${APP_NAME}:${imageTag}"
                    
                    echo "Building Docker image: ${imageName}"
                    sh "docker build -t ${imageName} ."
                    sh "docker push ${imageName}"
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                echo 'Deploying to staging environment'
                sh 'kubectl apply -f k8s/staging/'
            }
        }
        
        stage('Deploy to Production') {
            when {
                allOf {
                    branch 'main'
                    params.ENVIRONMENT == 'production'
                }
            }
            steps {
                input message: 'Deploy to production?',
                      ok: 'Deploy',
                      submitter: 'admin,manager'
                      
                echo 'Deploying to production environment'
                sh 'kubectl apply -f k8s/production/'
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace'
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
            slackSend channel: '#deployments',
                      color: 'good',
                      message: "✅ ${APP_NAME} build ${env.BUILD_NUMBER} succeeded"
        }
        failure {
            echo 'Pipeline failed!'
            slackSend channel: '#deployments',
                      color: 'danger',
                      message: "❌ ${APP_NAME} build ${env.BUILD_NUMBER} failed"
        }
    }
}
```

This example demonstrates:
- Environment variables for configuration
- Parameters for user input
- Tools for automatic installation
- Options for pipeline behavior
- Multiple stages with different purposes
- Parallel execution for tests
- When conditions for conditional deployment
- Script blocks for custom logic
- Input steps for manual approval
- Post actions for cleanup and notifications

This documentation covers the essential concepts and features of Jenkins declarative pipelines with clear explanations and practical examples. 