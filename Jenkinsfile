pipeline {
    agent any

    tools {
        maven 'Maven3'
        nodejs 'NodeJS'
    }
    
    triggers {
        pollSCM('* * * * *')
    }

    environment {
        JWT_SECRET = credentials('jwt-secret')
        INTERNAL_TOKEN = credentials('internal-token')
    }

    stages {
        
        stage('Checkout Git') {
            steps {
                echo 'Git Checkout in Progress...'
                git branch: 'main', url: 'https://github.com/Dahreau/buy-01.git'
        sh 'ls backend'
        sh 'ls frontend'
            }
        }
        
        stage('Build & Test Backend') {
            stages {
                stage('User Service') {
                    steps {
                        dir('backend/user-service') {
                            sh 'mvn clean test'
                        }
                    }
                }
                stage('Product Service') {
                    steps {
                        dir('backend/product-service') {
                            sh 'mvn clean test'
                        }
                    }
                }
                stage('Media Service') {
                    steps {
                        dir('backend/media-service') {
                            sh 'mvn clean test'
                        }
                    }
                }
            }
        }
        
        stage('Build & Test Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                    sh 'export CI=true && npm test -- --project=buy-frontend'
                }
            }
        }
    
        stage('Deploy with Rollback Strategy') {
            steps {
                script {
                    echo '🚀 Starting deployment process...'
                    
                    sh '''
                        docker compose pull || true
                        docker images --format "{{.Repository}}:{{.Tag}}" | grep buy-01 | grep latest > /tmp/current_images.txt || true
                        for img in $(cat /tmp/current_images.txt); do
                            docker tag $img ${img}-backup || true
                        done
                    '''
                    
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh '''
                            docker-compose build
                            docker-compose up -d
                            
                            sleep 15
                            
                            if docker ps | grep "Restarting\\|Exited" | grep "buy-01"; then
                                exit 1
                            fi
                        '''
                    }
                    
                    if (currentBuild.currentResult == 'FAILURE') {
                        sh '''
                            for img in $(docker images --format "{{.Repository}}:{{.Tag}}" | grep backup); do
                                original=$(echo $img | sed 's/-backup//')
                                docker tag $img $original || true
                            done
                            
                            docker-compose up -d
                        '''
                        error('Deployment failed. Rollback was executed.')
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "📊 Pipeline execution finished. Status: ${currentBuild.currentResult}"
            junit '**/target/surefire-reports/*.xml'
        }
        success {
            echo '🎉 All stages completed successfully!'
            mail(
                to: 'darosamakypro@gmail.com',
                subject: "✅ SUCCESS: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                    Build réussi !
                    Job: ${env.JOB_NAME}
                    Build #: ${env.BUILD_NUMBER}
                    URL: ${env.BUILD_URL}
                    Durée: ${currentBuild.durationString}
                """
            )
        }
        failure {
            echo '❌ Pipeline failed! Check logs above.'
            mail(
                to: 'darosamakypro@gmail.com',
                subject: "❌ FAILURE: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                body: """
                    Build échoué !
                    Job: ${env.JOB_NAME}
                    Build #: ${env.BUILD_NUMBER}
                    URL: ${env.BUILD_URL}
                    État: ${currentBuild.currentResult}
                    ⚠️ Action requise: Vérifiez les logs du build.
                """
            )
        }

    }
}