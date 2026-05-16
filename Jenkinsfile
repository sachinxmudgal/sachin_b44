pipeline {
    agent {
        label 'ec2'
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'test', 'staging', 'prod'],
            description: 'Select deployment environment'
        )
    }

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-8-openjdk-amd64"
        PATH = "/usr/local/bin:${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Verify Slave') {
            steps {
                sh 'hostname'
                sh 'whoami'
                sh 'java -version'
                sh 'mvn -version'
                sh 'gitleaks version || true'
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/opstree/spring3hibernate.git'
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

        stage('Compile and Unit Test') {
            parallel {

                stage('Compile') {
                    steps {
                        sh 'mvn compile -Dfindbugs.skip=true'
                    }
                }

                stage('Unit Test') {
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
                    ok: "Approve Build"
                )
            }
        }

        stage('Build') {
            steps {
                echo "Building for ${params.ENVIRONMENT}"

                sh """
                mvn package \
                -Dfindbugs.skip=true \
                -Denv=${params.ENVIRONMENT}
                """
            }
        }

        stage('Git Push') {
            steps {
                sh '''
                git config user.name "Jenkins"
                git config user.email "jenkins@example.com"

                echo "Build for ${ENVIRONMENT} completed $(date)" >> build.log

                git add build.log
                git commit -m "Auto build update" || true
                git push origin master || true
                '''
            }
        }
    }

    post {

        always {
            archiveArtifacts artifacts: 'gitleaks-report.json',
                             allowEmptyArchive: true

            archiveArtifacts artifacts: 'target/*.war',
                             allowEmptyArchive: true

            cleanWs()
        }

        success {
            echo "Pipeline completed successfully for ${params.ENVIRONMENT}"
        }

        failure {
            echo "Pipeline failed"
        }
    }
}
