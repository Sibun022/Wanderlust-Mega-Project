@Library('Shared') _

pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        SONAR_HOME = tool 'Sonar'
    }

    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Frontend Docker Image Tag')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Backend Docker Image Tag')
    }

    stages {

        stage('Validate Parameters') {
            steps {
                script {
                    if (!params.FRONTEND_DOCKER_TAG?.trim()) {
                        error("FRONTEND_DOCKER_TAG is required.")
                    }

                    if (!params.BACKEND_DOCKER_TAG?.trim()) {
                        error("BACKEND_DOCKER_TAG is required.")
                    }
                }
            }
        }

        stage('Workspace Cleanup') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                script {
                    clone(
                        "https://github.com/Sibun022/Wanderlust-Mega-Project.git",
                        "main"
                    )
                }
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                script {
                    trivy_scan()
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                script {
                    owasp_dependency()
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    sonarqube_analysis(
                        "Sonar",
                        "wanderlust",
                        "wanderlust"
                    )
                }
            }
        }

     stage('Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: false
        }
    }
}

        stage('Export Environment Variables') {
            parallel {

                stage('Backend') {
                    steps {
                        dir('Automations') {
                            sh 'bash updatebackendnew.sh'
                        }
                    }
                }

                stage('Frontend') {
                    steps {
                        dir('Automations') {
                            sh 'bash updatefrontendnew.sh'
                        }
                    }
                }

            }
        }

        stage('Verify Docker') {
            steps {
                sh '''
                    docker --version
                    docker ps
                '''
            }
        }

        stage('Docker Build') {
            steps {
                script {

                    dir('backend') {
                        docker_build(
                            imageName: 'biswa022/wanderlust-backend-beta',
                            imageTag: params.BACKEND_DOCKER_TAG
                        )
                    }

                    dir('frontend') {
                        docker_build(
                            imageName: 'biswa022/wanderlust-frontend-beta',
                            imageTag: params.FRONTEND_DOCKER_TAG
                        )
                    }

                }
            }
        }

        stage('Docker Push') {
            steps {
                script {

                    docker_push(
                        imageName: 'biswa022/wanderlust-backend-beta',
                        imageTag: params.BACKEND_DOCKER_TAG
                    )

                    docker_push(
                        imageName: 'biswa022/wanderlust-frontend-beta',
                        imageTag: params.FRONTEND_DOCKER_TAG
                    )

                }
            }
        }
    }

post {

    success {
        build job: 'Wanderlust-CD',
        parameters: [
            string(name: 'FRONTEND_DOCKER_TAG', value: params.FRONTEND_DOCKER_TAG),
            string(name: 'BACKEND_DOCKER_TAG', value: params.BACKEND_DOCKER_TAG)
        ]
    }

    failure {
        echo "Pipeline Failed"
    }

    always {
        cleanWs()
    }
}
}
