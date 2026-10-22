```groovy
pipeline {
    agent any

    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['test', 'production'],
            description: 'Select the deployment environment'
        )

        booleanParam(
            name: 'RUN_DEPLOYMENT',
            defaultValue: false,
            description: 'Run the Kubernetes deployment stage'
        )
    }

    environment {
        IMAGE_NAME = 'YOUR_DOCKERHUB_USERNAME/smart-agriculture-portal'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Application') {
            parallel {
                stage('HTML Validation') {
                    steps {
                        script {
                            if (!fileExists('index.html')) {
                                error('index.html is missing')
                            }

                            echo 'HTML file exists'
                        }
                    }
                }

                stage('Dockerfile Validation') {
                    steps {
                        script {
                            if (!fileExists('Dockerfile')) {
                                error('Dockerfile is missing')
                            }

                            echo 'Dockerfile exists'
                        }
                    }
                }

                stage('Kubernetes Validation') {
                    steps {
                        script {
                            if (!fileExists('deployment.yaml')) {
                                error('deployment.yaml is missing')
                            }

                            echo 'Kubernetes manifest exists'
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building the application image'
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Archive Build Artifacts') {
            steps {
                archiveArtifacts(
                    artifacts: 'index.html,Dockerfile,Jenkinsfile,deployment.yaml',
                    fingerprint: true
                )
            }
        }

        stage('Conditional Deployment') {
            when {
                expression {
                    return params.RUN_DEPLOYMENT
                }
            }

            steps {
                echo "Deployment requested for ${params.DEPLOY_ENV}"

                echo 'Configure Kubernetes deployment credentials before enabling this stage.'

                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }

    post {
        success {
            echo 'Smart Agriculture pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed. Review the console output.'
        }

        always {
            echo 'Pipeline execution finished'
        }
    }
}
```
