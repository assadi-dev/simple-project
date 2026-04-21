pipeline {

    agent {
        docker {
            image 'node:20-alpine'
            args  '-u root'
        }
    }

    environment {
        APP_NAME      = 'mon-app-next'
        COOLIFY_TOKEN = credentials('coolify-token')  // Jenkins Credentials → Secret Text
        COOLIFY_URL   = 'https://TON_COOLIFY'         // ex: https://coolify.mondomaine.com
        COOLIFY_UUID  = 'TON_APP_UUID'                // Coolify → App → Settings → UUID
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    stages {

        // ── 1. CHECKOUT ──────────────────────────────────────
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ── 2. INSTALL ───────────────────────────────────────
        stage('Install') {
            steps {
                sh '''
                    node -v && npm -v
                    npm ci --prefer-offline
                '''
            }
        }

        // ── 3. LINT ──────────────────────────────────────────
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        // ── 4. TYPE CHECK ────────────────────────────────────
        stage('Type Check') {
            steps {
                sh 'npx tsc --noEmit'
            }
        }

        // ── 5. TESTS ─────────────────────────────────────────
        stage('Tests') {
            steps {
                sh 'npm test -- --ci --passWithNoTests'
            }
        }

        // ── 6. BUILD ─────────────────────────────────────────
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        // ── 7. DEPLOY → COOLIFY WEBHOOK ──────────────────────
        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh '''
                    curl -f -X GET \
                        "${COOLIFY_URL}/api/v1/deploy?uuid=${COOLIFY_UUID}&force=false" \
                        -H "Authorization: Bearer ${COOLIFY_TOKEN}" \
                        -H "Content-Type: application/json"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline réussi — déploiement Coolify déclenché !'
        }
        failure {
            echo '❌ Pipeline échoué — Coolify non déclenché.'
        }
        always {
            cleanWs()
        }
    }
}