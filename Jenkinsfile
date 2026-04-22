pipeline {
    agent any

    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ACTION', value: '$.action']
            ],
            causeString: 'Triggered by Webhook from mattermost fork repo',
            token: 'mattermost-webhook'
        )
    }

    environment {
        REPO_OWNER = "zr0x8"
        REPO_NAME  = "mattermost"
        DEPLOY_DIR = "/opt/mattermost"
        ANSIBLE_HOST_KEY_CHECKING = "False"
    }

    stages {
        stage('Fetch Latest Tag') {
            steps {
                script {
                    def getTag = """
                        curl -s https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/releases/latest | \
                        jq -r '.tag_name'
                    """.trim()

                    env.LATEST_TAG = sh(script: getTag, returnStdout: true).trim()

                    if (env.LATEST_TAG == "null" || env.LATEST_TAG == "") {
                        error "Không tìm thấy release tag phù hợp trên GitHub!"
                    }
                    echo "Found latest tag: ${env.LATEST_TAG}"
                }
            }
        }

        stage('Deploy via Ansible') {
            steps {
                withCredentials([string(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS')]) {
                    sh '''
                        echo "$VAULT_PASS" > .vault_pass.txt
                        ansible-playbook -i ansible/inventory/hosts.ini \
                            --vault-password-file .vault_pass.txt \
                            -e "image_tag=${LATEST_TAG}" \
                            ansible/site.yml
                    '''
                }
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
