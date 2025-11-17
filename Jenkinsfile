pipeline {
    agent any

    tools {
        maven '3.9.11'      // must match the Maven name in Jenkins -> Manage Jenkins -> Tools
    }

    environment {
        // OLD
        // NEXUS_URL   = 'http://nexus:8081'

        // NEW
        NEXUS_URL   = 'http://host.docker.internal:8081'
        NEXUS_REPO  = 'maven-releases'
        GROUP_ID    = 'com.example'
        ARTIFACT_ID = 'spring-boot-complete'
        VERSION     = '0.0.1-SNAPSHOT'
        NEXUS_CREDS = 'nexus-cred-id'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/angelovskid1/gs-spring-boot3.git'
            }
        }

        stage('Test') {
            steps {
                sh 'git --version'
                sh 'mvn --version'
                sh 'mvn clean test'
            }
        }

        stage('Build and Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Upload to Nexus') {
            steps {
                script {
                    // The JAR Jenkins just built
                    def jarFile = "target/${ARTIFACT_ID}-${VERSION}.jar"

                    echo "Uploading ${jarFile} to Nexus..."

                    withCredentials([usernamePassword(
                        credentialsId: NEXUS_CREDS,
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )]) {
                        sh """
                            curl -v -u $NEXUS_USER:$NEXUS_PASS \
                              --upload-file ${jarFile} \
                              ${NEXUS_URL}/repository/${NEXUS_REPO}/${GROUP_ID.replace('.', '/')}/${ARTIFACT_ID}/${VERSION}/${ARTIFACT_ID}-${VERSION}.jar
                        """
                    }
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Build successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}

