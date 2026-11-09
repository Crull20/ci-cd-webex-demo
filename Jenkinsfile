pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh '''#!/bin/bash
set -e
python3 -m venv .venv
. .venv/bin/activate
python3 -m pip install -r requirements.txt
bash build.sh
'''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '''#!/bin/bash
set -e
. .venv/bin/activate
bash test.sh
'''
            }
        }
    }

    post {
        always {
            withCredentials([
                string(credentialsId: 'webex-bot-token', variable: 'WEBEX_BOT_TOKEN'),
                string(credentialsId: 'webex-room-id', variable: 'WEBEX_ROOM_ID')
            ]) {
                withEnv(["BUILD_RESULT=${currentBuild.currentResult}"]) {
                    sh '''#!/bin/bash
set -e
python3 - <<'PY'
import json
import os
import urllib.request

status = os.environ['BUILD_RESULT']
job = os.environ.get('JOB_NAME', 'Jenkins pipeline')
number = os.environ.get('BUILD_NUMBER', '?')
message = f"**Jenkins Build: {status}**\\nJob: {job}\\nBuild: #{number}"
payload = {
    'roomId': os.environ['WEBEX_ROOM_ID'],
    'markdown': message,
}
request = urllib.request.Request(
    'https://webexapis.com/v1/messages',
    data=json.dumps(payload).encode('utf-8'),
    headers={
        'Authorization': 'Bearer ' + os.environ['WEBEX_BOT_TOKEN'],
        'Content-Type': 'application/json',
    },
    method='POST',
)
with urllib.request.urlopen(request, timeout=20) as response:
    print('Webex notification sent. HTTP status:', response.status)
PY
'''
                }
            }
        }
    }
}
