pipeline {
    agent any

    stages {

        stage('Git Config') {
            steps {
                sh '''
                    git config --global user.name "Nithin Gowda"
                    git config --global user.email "nithingowdai46@gmail.com"
                '''
            }
        }

        stage('Clone Application') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/NithinGowda46/Project2-chatapp-CI.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                sh '''
                    docker build --no-cache \
                    -t chat-app-backend:${BUILD_NUMBER} \
                    ./backend
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                    docker build --no-cache \
                    -t chat-app-frontend:${BUILD_NUMBER} \
                    ./frontend
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password --region ap-southeast-1 | \
                    docker login --username AWS --password-stdin \
                    236209348145.dkr.ecr.ap-southeast-1.amazonaws.com
                '''
            }
        }

        stage('Tag Backend Image') {
            steps {
                sh '''
                    docker tag \
                    chat-app-backend:${BUILD_NUMBER} \
                    236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-backend:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Backend Image') {
            steps {
                sh '''
                    docker push \
                    236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-backend:${BUILD_NUMBER}
                '''
            }
        }

        stage('Tag Frontend Image') {
            steps {
                sh '''
                    docker tag \
                    chat-app-frontend:${BUILD_NUMBER} \
                    236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-frontend:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh '''
                    docker push \
                    236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-frontend:${BUILD_NUMBER}
                '''
            }
        }

        stage('Clone CD Repository') {
            steps {
                dir('cd-repo') {
                    git branch: 'main',
                        credentialsId: 'github-ssh',
                        url: 'git@github.com:NithinGowda46/Project2-chatapp-CD.git'
                }
            }
        }

        stage('Update Backend Image') {
            steps {
                dir('cd-repo') {
                    sh '''
                        sed -i "s|image: 236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-backend:.*|image: 236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-backend:${BUILD_NUMBER}|" k8s/backend_deployment.yaml
                    '''
                }
            }
        }

        stage('Update Frontend Image') {
            steps {
                dir('cd-repo') {
                    sh '''
                        sed -i "s|image: 236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-frontend:.*|image: 236209348145.dkr.ecr.ap-southeast-1.amazonaws.com/chat-app-frontend:${BUILD_NUMBER}|" k8s/frontend_deployment.yaml
                    '''
                }
            }
        }

        stage('Git Commit') {
            steps {
                dir('cd-repo') {
                    sh '''
                        git add k8s/backend_deployment.yaml k8s/frontend_deployment.yaml && \
                        git commit -m "Update backend and frontend images to build ${BUILD_NUMBER}" || \
                        echo "No changes to commit"
                    '''
                }
            }
        }

        stage('Git Push') {
            steps {
                dir('cd-repo') {
                    sh '''
                        git push origin main
                    '''
                }
            }
        }
    }
}