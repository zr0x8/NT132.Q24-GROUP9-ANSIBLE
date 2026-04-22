pipeline {
    agent any

    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ACTION', value: '$.action']
            ],
            causeString: 'Triggered by Webhook from Repo A',
            token: 'mattermost-webhook'
        )
    }

    environment {
        REPO_OWNER = "zr0x8"
        REPO_NAME  = "mattermost"
        DEPLOY_DIR = "/opt/mattermost"
    }

    stages {
        stage('Fetch Latest Artifact URL') {
            steps {
                script {
                    def getUrl = """
                        curl -s https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/releases/latest | \
                        jq -r '.assets[] | select(.name | contains("linux-amd64.tar.gz")) | .browser_download_url'
                    """.trim()

                    env.LATEST_URL = sh(script: getUrl, returnStdout: true).trim()

                    if (env.LATEST_URL == "null" || env.LATEST_URL == "") {
                        error "Không tìm thấy file artifact phù hợp trên GitHub Release!"
                    }
                    echo "Found latest artifact: ${env.LATEST_URL}"
                }
            }
        }

        stage('Pull artifact') {
            steps {
                sh "wget -O mattermost.tar.gz ${env.LATEST_URL}"
            }
        }

        stage('Deploy via Ansible') {
            steps {
                sh '''
                    ansible-playbook -i ansible/inventory/hosts.ini \
                        -e "artifact_path=${WORKSPACE}/mattermost.tar.gz" \
                        -e "deploy_dir=${DEPLOY_DIR}" \
                        ansible/site.yml
                '''
            }
        }
    }

    post {
        always {
            deleteDir()
        }
        success { echo 'Deploy thành công phiên bản mới nhất!' }
        failure { echo 'Deploy thất bại, kiểm tra log API hoặc kết nối Ansible.' }
    }
}
