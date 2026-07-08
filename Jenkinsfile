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