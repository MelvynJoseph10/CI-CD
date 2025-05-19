pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
    APP_NAME     = "register-app-pipeline"
    RELEASE      = "1.0.0"
    DOCKER_USER  = "melvynjoseph17"
    DOCKER_PASS  = 'dockerhub'
    IMAGE_NAME   = "${DOCKER_USER}/${APP_NAME}"
    IMAGE_TAG    = "${RELEASE}-${BUILD_NUMBER}"
}

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/MelvynJoseph10/CI-CD.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

stage("Build & Push Docker Image") {
    steps {
        script {
            // Ensure the WAR file is present
            sh '''
                if ls webapp/target/*.war 1> /dev/null 2>&1; then
                    cp webapp/target/*.war .
                else
                    echo "ERROR: WAR file not found in webapp/target/"
                    exit 1
                fi
            '''

            // Build and push Docker image
            docker.withRegistry('', DOCKER_PASS) {
                def docker_image = docker.build("${IMAGE_NAME}")
                docker_image.push("${IMAGE_TAG}")
                docker_image.push('latest')
            }
        }
    }
}



    }
}
