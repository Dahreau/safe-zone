pipeline {
    agent any

    tools {
        maven 'Maven3'
    }
    
    triggers {
        pollSCM('* * * * *')
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
                        echo "1. Creating backup of current version..."
                        echo "2. Deploying new version to staging environment..."
                        echo "3. Running health check..."
                        
                        # --- POINT CRITIQUE : SIMULE UN ÉCHEC ALÉATOIRE POUR MONTRER LE ROLLBACK ---
                        # Pour tester le rollback, décommente la ligne suivante :
                        # echo "❌ Health check failed!" && exit 1
                        
                        echo "✅ Health check passed."
                        echo "4. Switching traffic to new version..."
                    '''
                    
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        sh '''
                            # Si une des étapes précédentes échoue, ce bloc ne s'exécute pas
                            echo "✅ Deployment completed successfully."
                        '''
                    }
                    
                    if (currentBuild.currentResult == 'FAILURE') {
                        echo '🔄 Initiating rollback procedure...'
                        sh '''
                            echo "1. Rolling back to previous stable version..."
                            echo "2. Restoring from backup..."
                            echo "3. Verifying system is operational..."
                            echo "✅ Rollback completed. System is stable."
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