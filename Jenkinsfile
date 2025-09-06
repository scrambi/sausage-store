pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *') // запуск каждые ~5 минут
    }

    tools {
        maven 'maven-3.9.11'
        jdk 'jdk16'
        nodejs 'node16'
    }

    stages {
        stage('Build & Test backend') {
            steps {
                dir("backend") {
                    sh 'mvn -B clean package'
                }
            }
            post {
                success {
                    junit 'backend/target/surefire-reports/**/*.xml'
                }
            }
        }

        stage('Build frontend') {
            steps {
                script {
                    def nodeHome = tool 'node16'
                    withEnv(["PATH+NODE=${nodeHome}/bin"]) {
                        dir("frontend") {
                            sh '''
                                set -e
                                node -v
                                npm -v
                                npm ci || npm install --legacy-peer-deps
                                npm run build -- --prod || npm run build
                            '''
                        }
                    }
                }
            }
        }

        stage('Save artifacts') {
            steps {
                archiveArtifacts(artifacts: 'backend/target/sausage-store-0.0.1-SNAPSHOT.jar')
                archiveArtifacts(artifacts: 'frontend/dist/frontend/*')
            }
            post {
                success {
                    withCredentials([string(credentialsId: 'telegram-bot-token', variable: 'TELEGRAM_TOKEN')]) {
                        sh '''
                            curl -s -X POST -H 'Content-Type: application/json' \
                            --data "{\\"chat_id\\": \\"180424264\\", \\"text\\": \\"Sergey собрал приложение успешно ✅\\"}" \
                            https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage
                        '''
                    }
                }
            }
        }
    }
}
