pipeline {
  agent any

  tools {
    jdk    'jdk16'
    maven  'maven-3.9.11'
    nodejs 'node16'   // оставляем 16; при желании можно вернуть node14
  }

  options { timestamps() /* ansiColor('xterm') — включай, если стоит AnsiColor */ }

  stages {
    stage('Build & Test backend') {
      steps {
        dir('backend') {
          sh 'mvn -B clean package'
        }
      }
      post {
        always {
          junit 'backend/target/surefire-reports/**/*.xml'
        }
      }
    }

    stage('Build frontend') {
      steps {
        dir('frontend') {
          sh '''
            set -e
            node -v
            npm -v || true
            npm i -g npm@8
            npm -v
            npm ci || npm install --legacy-peer-deps
            npm run build -- --prod || npm run build
          '''
        }
      }
    }

    stage('Save artifacts') {
      steps {
        // jar может менять имя — берём маской
        archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
        // тут лежит сборка Angular — без лишней вложенной папки
        archiveArtifacts artifacts: 'frontend/dist/**',      fingerprint: true
      }
      post {
        success {
          // Telegram
          withCredentials([
            string(credentialsId: 'telegram-bot-token', variable: 'BOT_TOKEN'),
            string(credentialsId: 'telegram-chat-id',  variable: 'CHAT_ID')
          ]) {
            sh '''
              SHORT_SHA="${GIT_COMMIT:0:7}"
              MSG="✅ Сборка успешна: $JOB_NAME #$BUILD_NUMBER (commit ${SHORT_SHA})"
              curl -sS -X POST -H "Content-Type: application/json" \
                   --data "{\"chat_id\":\"$CHAT_ID\",\"text\":\"$MSG\"}" \
                   https://api.telegram.org/bot$BOT_TOKEN/sendMessage || true
            '''
          }

          // Chuck Norris (не валим билд, если плагин не установлен)
          script {
            try { chuckNorris() } catch (ignored) { echo 'Chuck Norris plugin not available' }
          }
        }
      }
    }
  }
}
