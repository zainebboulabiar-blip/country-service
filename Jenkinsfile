pipeline {
    agent any
    
    tools {
        maven 'mvn'
    }
    
    stages {
        stage('Checkout code') {
            steps {
                git branch: 'main', url: 'https://github.com/zainebboulabiar-blip/country-service.git'
            }
        }
        
        stage('Compile, test code, package in war file and store in maven repo') {
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                    junit allowEmptyResults: true,
                        testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Dockerfile') {
            steps {
                sh 'docker build . -t my-country-service:$BUILD_NUMBER'
                withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u raedbourouis -p ${dockerhubpwd}'
                }
                sh 'docker tag my-country-service:$BUILD_NUMBER zainebboulabiar-blip/my-country-service:$BUILD_NUMBER'
                sh 'docker push zainebboulabiar/my-country-service:$BUILD_NUMBER'
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubeconfig(credentialsId: 'kubeconfig-file', serverUrl: '') {
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl apply -f service.yaml'
                    }
                }
            }
        }

       
    }
    
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check console output for details.'
        }
    }
}
