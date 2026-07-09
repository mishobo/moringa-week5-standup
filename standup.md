## Stand up Content
1. CI-CD definition and principles - ELizabeth
2. setup tools jenkins, docker, conneting to GIT basic pipeline : Mwikali
3. advanced pipeline: vm deployment - Hussein


### CI-CD definition and principles
// Add content here


### setup tools jenkins, docker, conneting to GIT basic pipeline
// Add content here

### advanced pipeline: vm deployment
## Step 1 create a Jenkinsfile template
```groovy
    pipeline {
        agent any       
        stages {
            stage('Clone repository') {
                steps { 
                    echo "Cloning repository ..."
                }  
            }
            stage('Build') {
                steps { 
                    echo "Building application ..."
                }  
            }
            stage('Test') {
                steps {
                    echo "Testing application ..."
                }
            }
            stage('Package') {
                steps {
                    echo "Packaging application ..."
                }
            }
            stage('Containerize') {
                steps {
                    echo "Containerizing application ...."
                }
            }
            stage('Deploy') {
                steps {
                    echo "Deploy deploying application ..."
                }
            }
        }
        post { 
            success {
                echo "Notify team pipeline was successful"
            }
            failure {
                echo "Notify team pipeline failed"
            }
            always {
                echo "Cleanup workspace"
            }
        }            
    }
```

## Step 2: create a jenkins pipeline on the UI
   - New item
   - item type pipeline: java-todo-pipeline
   - configuration: 
      - pipeline: Pipeline from script from SCM
      - SCM: GIT
      - Repository URL: https://github.com/mishobo/java-todo
      - Branch Specifier: master
      - Apply & save
      - run build pipeline manually to test

## Step 3: Cloning stage syntax
```groovy
    stage('Checkout') {
        steps {
            checkout scm
            script {
                env.GIT_COMMIT_SHORT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                def branch = (env.BRANCH_NAME ?: env.GIT_BRANCH ?: 'unknown').replaceFirst(/^origin\//, '')
                env.GIT_BRANCH_NAME = branch
                env.IS_RELEASE_BRANCH = (branch == 'main' || branch == 'master').toString()
                env.IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT_SHORT}"
            }
            echo "Building ${env.GIT_BRANCH_NAME} @ ${env.GIT_COMMIT_SHORT} as ${env.IMAGE_NAME}:${env.IMAGE_TAG} (release branch: ${env.IS_RELEASE_BRANCH})"
        }
    }
```

## Step 4: Building stage
```groovy
    stage('Build') {
        steps {
            sh './gradlew clean build -x test --no-daemon'
        }
    }
```

## Step 5: Run application's Tests stage
```groovy
    stage('Test') {
        steps {
            sh './gradlew test --no-daemon'
        }
        post {
            always {
                junit testResults: 'build/test-results/test/*.xml', allowEmptyResults: true
            }
        }
    }
```

## Step 6: Package application

```groovy
    stage('Package') {
        steps {
            sh './gradlew installDist -x test --no-daemon'
            archiveArtifacts artifacts: 'build/install/java-todo/**', fingerprint: true
        }
    }
```   

## 7: Configure environment variables
```groovy
    environment {
        IMAGE_NAME             = 'mishobo/todo-list-app'
        CONTAINER_NAME         = 'todo-app'
        APP_PORT               = '4567'
        DOCKERHUB_CREDENTIALS  = 'dockerhub-credentials'  // Jenkins Credentials ID: Docker Hub username/password (or access token)
        DEPLOY_SSH_CREDENTIALS = 'vm-deploy-ssh-key'      // Jenkins Credentials ID: SSH private key
        DEPLOY_USER            = 'deploy'
        NOTIFY_EMAIL           = 'team@example.com'       // TODO: replace with your real notification list
    }
```

## Step 8: Install docker service in jenkins container

##### Docker Compose setup 
```yaml
services:

  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    user: root
    restart: always
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  ubuntu_data:
  jenkins_home:
```

#### docker set up on jenkins
```bash
docker exec -u root  -it jenkins bash
apt update
apt upgrade
apt install docker.io
```

##### Dockerfile

```Dockerfile
# Runtime-only image: expects `./gradlew installDist` to have already produced
# build/install/java-todo/ (Jenkins' Package stage does this before `docker build`).
# Keeping compilation out of the image build keeps CI as the single source of
# truth for what got tested and keeps the image build fast and reproducible.
FROM eclipse-temurin:21-jre-alpine

RUN addgroup -S app && adduser -S -G app -h /app app

WORKDIR /app
COPY build/install/java-todo/ ./
RUN chown -R app:app /app

USER app
EXPOSE 4567

# Handlebars (via reflection) needs java.util opened up under the JDK 9+
# module system, otherwise every route throws InaccessibleObjectException.
ENV JDK_JAVA_OPTIONS="--add-opens java.base/java.util=ALL-UNNAMED"

HEALTHCHECK --interval=30s --timeout=3s --start-period=15s --retries=3 \
    CMD wget -qO- http://127.0.0.1:4567/ >/dev/null 2>&1 || exit 1

ENTRYPOINT ["./bin/java-todo"]
```

## Step 9: Containerizing the application
```groovy
    stage('Containerize') {
        steps {
            sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:build-${env.BUILD_NUMBER} ."
        }
    }
```        

## Step 10: Push conatiner to Docker registry
