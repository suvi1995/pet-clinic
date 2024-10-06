pipeline {
    agent any 
    
    tools{
        jdk 'jdk'
        maven 'Maven'
    }
    
    environment {
        registry = '897886641913.dkr.ecr.us-east-1.amazonaws.com/demo'
        KUBECONFIG_PATH = credentials('kubeconfig-file')
    }
    
    stages{
        
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/suvi1995/demoproject.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        
        stage("Build"){
            steps{
                sh "mvn clean install"
            }
        }
        stage('Docker Build') {
            steps {
               script {
                  dockerImage = docker.build registry 
                  dockerImage.tag("$BUILD_NUMBER")
              }
            }
        }
        
          stage('Docker Push') {
            steps {
                script {
                  sh'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 897886641913.dkr.ecr.us-east-1.amazonaws.com'
                  sh'docker build -t demo .'
                  sh 'docker tag demo:latest 897886641913.dkr.ecr.us-east-1.amazonaws.com/demo:latest'
                  sh'docker push 897886641913.dkr.ecr.us-east-1.amazonaws.com/demo:latest'
                }
               
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withEnv(["KUBECONFIG=${env.KUBECONFIG_PATH}"]) {
                        sh 'kubectl apply -f deployment.yaml'
                    }
                }  
            }
        }
        
    }
}
