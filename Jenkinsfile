pipeline {
    agent any
    triggers { githubPush() }

    stages {
        stage('Build') {
            steps { sh 'python3 -m py_compile sample_code.py' }
        }
        stage('Test') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install -q pytest
                    .venv/bin/python -m pytest -v tests/
                '''
            }
        }
    }

    post {
        always {
            script {
                withCredentials([
                    string(credentialsId: 'webex-token', variable: 'TOKEN'),
                    string(credentialsId: 'webex-room', variable: 'ROOM')
                ]) {
                    withEnv(["STATUS=${currentBuild.currentResult}"]) {
                        sh '''
                            set +x
                            curl -fsS https://webexapis.com/v1/messages \
                              -H "Authorization: Bearer $TOKEN" \
                              --data-urlencode "roomId=$ROOM" \
                              --data-urlencode "text=Jenkins build $BUILD_NUMBER: $STATUS" \
                              -o /dev/null
                            echo "Webex message sent"
                        '''
                    }
                }
            }
        }
    }
}
