pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'test', 'staging', 'prod'],
            description: 'Select deployment environment'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/opstree/spring3hibernate.git'
            }
        }

        stage('GitLeaks Scan') {
            steps {
                sh '''
                gitleaks detect \
                --source . \
                --report-format json \
                --report-path gitleaks-report.json \
                || true
                '''
            }
        }

        stage('Compile and Test') {
            environment {
                JAVA_HOME = "/usr/lib/jvm/java-8-openjdk-amd64"
                PATH = "${JAVA_HOME}/bin:${env.PATH}"
            }

            parallel {

                stage('Compile') {
                    steps {
                        sh 'mvn compile -Dfindbugs.skip=true'
                    }
                }

                stage('Test') {
                    steps {
                        sh 'mvn test -Dfindbugs.skip=true'
                    }
                }
            }
        }

        stage('Approval') {
            steps {
                input(
                    message: "Approve build for ${params.ENVIRONMENT} environment?",
                    ok: 'Proceed'
                )
            }
        }

        stage('Build') {
            environment {
                JAVA_HOME = "/usr/lib/jvm/java-8-openjdk-amd64"
                PATH = "${JAVA_HOME}/bin:${env.PATH}"
            }

            steps {
                echo "Building for ${params.ENVIRONMENT}"

                sh """
                mvn package \
                -Dfindbugs.skip=true \
                -Denv=${params.ENVIRONMENT}
                """
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
            archiveArtifacts artifacts: 'target/*.war', allowEmptyArchive: true
        }

        success {
            echo "Pipeline completed for ${params.ENVIRONMENT}"
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}
