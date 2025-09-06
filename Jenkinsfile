pipeline {
  agent any

  // В multibranch это триггерит только рескан SCM, не автоповтор билда без коммита
  triggers { pollSCM('H/5 * * * *') }

  tools {
    maven  'maven-3.9.11'
    jdk    'jdk16'
    nodejs 'node16'
  }

  stages {

    stage('Build & Test backend') {
      steps {
        dir('backend') {
          sh 'mvn -B clean package'
        }
      }
      post {
        success {
          junit 'backend/target/1surefire-reports/**/*.xml'
        }
        failure {
          withCredentials([string(credentialsId: 'telegram-bot-token', variable: 'TELEGRAM_TOKEN')]) {
            sh '''
              curl -s -X POST -H 'Content-Type: application/json' \
              --data "{\"chat_id\":\"180424264\",\"text\":\"❌ Упала стадия *Build & Test backend* в $JOB_NAME #$BUILD_NUMBER\"}" \
              https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage || true
            '''
          }
        }
      }
    }

    stage('Build frontend') {
      steps {
        script {
          def nodeHome = tool 'node16'
          withEnv(["PATH+NODE=${nodeHome}/bin"]) {
            dir('frontend') {
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
      // При желании можно уведомлять и об ошибке фронта:
      // post {
      //   failure {
      //     withCredentials([string(credentialsId: 'telegram-bot-token', variable: 'TELEGRAM_TOKEN')]) {
      //       sh '''
      //         curl -s -X POST -H 'Content-Type: application/json' \
      //         --data "{\"chat_id\":\"180424264\",\"text\":\"❌ Упала стадия *Build frontend* в $JOB_NAME #$BUILD_NUMBER\"}" \
      //         https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage || true
      //       '''
      //     }
      //   }
      // }
    }

    stage('Save artifacts') {
      steps {
        archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
        archiveArtifacts artifacts: 'frontend/dist/**',      fingerprint: true
      }
      post {
        success {
          withCredentials([string(credentialsId: 'telegram-bot-token', variable: 'TELEGRAM_TOKEN')]) {
            sh '''
              curl -s -X POST -H 'Content-Type: application/json' \
              --data "{\"chat_id\":\"180424264\",\"text\":\"✅ Сборка успешна: $JOB_NAME #$BUILD_NUMBER\"}" \
              https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage || true
            '''
          }
        }
      }
    }

  }
}
