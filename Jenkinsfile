pipeline {
    agent {
        label 'ec2'
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

        stage('Build') {
            steps {
                sh 'mvn package -Dfindbugs.skip=true'
            }
        }

        stage('Git Push') {
            steps {
                sh '''
                git config user.name "Jenkins"
                git config user.email "jenkins@example.com"

                echo "Build completed $(date)" >> build.log

                git add build.log
                git commit -m "Auto build update" || true
                git push origin master || true
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
            archiveArtifacts artifacts: 'target/*.war', allowEmptyArchive: true
            cleanWs()
        }

        success {
            echo 'Pipeline succeeded on EC2 slave'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}
