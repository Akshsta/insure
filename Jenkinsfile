pipeline {
    agent any

    environment {
        IMAGE_NAME = "akku1999/insure-me"
        IMAGE_TAG = "3.0"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials' // Jenkins credential ID
        SSH_CREDENTIALS_ID = 'ubuntu'           // Jenkins credential ID
        EC2_HOST = '44.205.248.180'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Akshsta/insure.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Generate HTML Report') {
            steps {
                sh 'mvn test -DsuiteXmlFile=testng.xml'
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'target/surefire-reports',
                    reportFiles: 'emailable-report.html',
                    reportName: 'TestNG Report'
                ])
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Test Server (via Ansible)') {
            steps {
                sshagent(credentials: ['ubuntu']) {
                    sh """
                    ansible-playbook -i inventory.ini ansible-playbook.yml
                    """
                }
            }
        }

        stage('Promote to Prod') {
            steps {
                sshagent([SSH_CREDENTIALS_ID]) {
                    sh """
                    ansible-playbook -i inventory.ini ansible-playbook.yml --tags prod
                    """
                }
            }
        }
    }

    post {
        failure {
            mail to: 'akshatafansekar17@gmail.com',
                 subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Check Jenkins for details: ${env.BUILD_URL}"
        }
    }
}
