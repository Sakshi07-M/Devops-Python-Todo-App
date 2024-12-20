pipeline{
    agent any
    tools{
        jdk 'jdk17'
    }

    environment {
        PYTHON = '/usr/bin/python3'
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }

        stage('Checkout From Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Sakshi07-M/Devops-Python-Todo-App.git'
            }
        }

        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Todo-App \
                    -Dsonar.projectKey=Todo-App '''
                }
            }
        }

        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
                       sh "docker build -t sakshi0721/todo-app:${BUILD_NUMBER} ."
                       sh "docker push sakshi0721/todo-app:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage("TRIVY Image Scan"){
            steps{
                sh "trivy image sakshi0721/todo-app:${BUILD_NUMBER} > trivy/trivyimage.txt" 
            }
        }

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "Devops-Python-Todo-App"
                GIT_USER_NAME = "Sakshi07-M"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        pwd
                        ls -lrt
                        git config user.email "majalekars2107@gmail.com"
                        git config user.name "Sakshi07-M"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        IMAGE_TAG=$(grep -oP '(?<=todo-app:)[^ ]+' kubernetes/deployment.yaml)  
                        
                        sed -i "s/todo-app:${IMAGE_TAG}/todo-app:${BUILD_NUMBER}/g" kubernetes/deployment.yaml                        
                        git add kubernetes/deployment.yaml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git add trivy/trivyimage.txt
                        git commit -m "Updated trivyimage.txt file"                        
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
       }
        
    }
}
