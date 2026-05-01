pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/zr0x8/NT132.Q24-GROUP9-ANSIBLE.git'
        BRANCH = 'main'
        INVENTORY_PATH = 'ansible/inventory/hosts.ini'
        VAULT_CREDENTIALS_ID = 'ansible-vault-pass'
        ANSIBLE_FORCE_COLOR = 'false'
        ANSIBLE_NOCOLOR = 'true'
        PLAYBOOK_MATTERMOST = 'ansible/site.yml'
        PLAYBOOK_ZABBIX = 'ansible/deploy_zabbix.yml'
        EXTRA_VARS_ALL = '-e @ansible/group_vars/all/secrets.yml -e @ansible/group_vars/all/secrets_zabbix.yml'
    }

    stages {
        stage('Checkout Repository') {
            steps {
                deleteDir()
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Preflight') {
            steps {
                sh '''
                    set -e
                    command -v ansible-playbook >/dev/null 2>&1
                    test -f "${INVENTORY_PATH}"
                '''
            }
        }

        stage('Syntax Check') {
            steps {
                sh '''
                    set -e
                    ansible-playbook -i "${INVENTORY_PATH}" "${PLAYBOOK_MATTERMOST}" --syntax-check
                    ansible-playbook -i "${INVENTORY_PATH}" "${PLAYBOOK_ZABBIX}" --syntax-check
                '''
            }
        }

        stage('Deploy via Ansible') {
            steps {
                withCredentials([string(credentialsId: "${VAULT_CREDENTIALS_ID}", variable: 'VAULT_PASS')]) {
                    sh '''
                        set -e
                        VAULT_FILE="${WORKSPACE}/.vault_pass"
                        printf '%s' "${VAULT_PASS}" > "${VAULT_FILE}"
                        chmod 600 "${VAULT_FILE}"
                        ansible-playbook -v -i "${INVENTORY_PATH}" "${PLAYBOOK_MATTERMOST}" ${EXTRA_VARS_ALL} --vault-password-file "${VAULT_FILE}"
                        ansible-playbook -v -i "${INVENTORY_PATH}" "${PLAYBOOK_ZABBIX}" ${EXTRA_VARS_ALL} --vault-password-file "${VAULT_FILE}"
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f "${WORKSPACE}/.vault_pass" || true'
            deleteDir()
        }
        success {
            echo 'Deploy thanh cong.'
        }
        failure {
            echo 'Deploy that bai. Kiem tra log Jenkins va Ansible output.'
        }
    }
}