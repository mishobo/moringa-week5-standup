## Stand up Content
1. CI-CD definition and principles - ELizabeth
2. setup tools jenkins, docker, conneting to GIT basic pipeline : Mwikali
3. advanced pipeline: vm deployment - Hussein


### CI-CD definition and principles
// Add content here


### setup tools jenkins, docker, conneting to GIT basic pipeline
// Add content here

### advanced pipeline: vm deployment
// Add content here
1. create jenkinsfile
2. create a skeleton
    ```groovy
    pipeline {
        agent any       
        stages {
            stage('Build') {
                steps { 
                    echo "Build stage: TODO"
                }  
            }
            stage('Test') {
                steps {
                    echo "Test stage: TODO"
                }
            }
            stage('Deploy') {
                steps {
                    echo "Deploy stage: TODO"
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
3. Setup Jenkins job
   - New item
   - item type pipeline: week5-standup
   - configuration: 
      - pipeline: Pipeline from script from SCM
      - SCM: GIT
      - Repository URL: https://github.com/mishobo/moringa-week5-standup
      - Branch Specifier: feat/vm-jenkinspipeline
      - Apply & save
      - run build pipeline manually to test