def dev_env='192.168.1.70';      
def qa_env='192.168.1.71';      //debian    
def prod_env='192.168.1.72';    //ubuntu
pipeline {
    agent {label 'DEV'}

    stages {
        stage('clone') {
            steps {
                dir('repositorio'){
                   git branch: 'server', url: 'https://github.com/Paola-Rios/flask-vue-crudUI'
                   //Se simula el repositorio de server con un branch para server
                }
                dir('frontend'){
                    git branch: 'client', url: 'https://github.com/Paola-Rios/flask-vue-crudUI'
                    //se simula el repositorio de client/ui con el branch client
                }
            }
        }
        stage ('docker build'){
            steps{
               dir('backend'){
                sh "docker build -t backend:1.0 ."

               }
               dir('frontend'){
                sh "docker build -t frontend:1.0 ."
               }
            }
        }
        stage('Deploy  DEV'){
            steps{
                sh "mkdir -p /home/ubuntu/Paola/Recuperatorio"
                dir ('/home/ubuntu/Paola/Recuperatorio'){
                   cleanWs()
                   git branch: 'main', url: 'https://github.com/Paola-Rios/RecuperatorioJenkinsFiles.git' 
                   sh "docker-compose rm -sf"
                   sh "docker-compose up -d"
                }
            }
        }
        stage('Testing DEV'){
            steps{
                sh "curl -v http://${dev_env}:5000/ | true" 
                sh "curl -v http://${dev_env}:8080/ | true"   // True por si no se conecta y no rompa el build
            }

        }
        stage('Docker save'){
            steps{
                sh "docker save frontend:1.0 | gzip > frontend.tar.gz"
                sh "docker save backend:1.0 | gzip > backend.tar.gz"
                stash name: 'backend', includes: 'backend.tar.gz'
                stash name: 'frontend', includes: 'frontend.tar.gz'
            }
        }
        stage('DeployQA'){
            agent{label 'qa'}
            steps{
                sh "mkdir -p /home/debian/Recuperatorio"
                dir ('/home/debian/Recuperatorio'){
                   cleanWs()
                   git branch: 'main', url: 'https://github.com/Paola-Rios/RecuperatorioJenkinsFiles.git' 
                   sh "docker-compose rm -sf"
                   sh "echo 'Y' | docker image prune -a"
                   unstash 'backend'
                   unstash 'frontend'
                   sh "docker load -i backend.tar.gz"
                   sh "docker load -i frontend.tar.gz"
                   sh "docker-compose up -d"
                }
            }

        }
        stage('Testing QA'){
            steps{
                sh "curl -v http://${qa_env}:5000/ | true" 
                sh "curl -v http://${qa_env}:8080/ | true" 
            }
        }
        stage('Deploy PROD'){
            agent{label 'prod'}
            steps{
                sh "mkdir -p /home/ubuntu/Recuperatorio"
                dir ('/home/ubuntu/Recuperatorio'){
                   cleanWs()
                   git branch: 'main', url: 'https://github.com/Paola-Rios/RecuperatorioJenkinsFiles.git' 
                   sh "docker compose rm -sf"
                   sh "echo 'Y' | docker image prune -a"
                   unstash 'backend'
                   unstash 'frontend'
                   sh "docker load -i backend.tar.gz"
                   sh "docker load -i frontend.tar.gz"
                   sh "docker compose up -d"
                }
            }
        }
        stage('Testing PROD'){
            steps{
                sh "curl -v http://${prod_env}:5000/ | true" //backend 
                sh "curl -v http://${prod_env}:8080/ | true" //frontend
            }
        }
    }
}
