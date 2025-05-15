pipeline {
    agent { label 'vagrant' }

    parameters {
        string(name: 'servers', defaultValue: '192.168.10.10', description: 'Server IPs')
        string(name: 'clients', defaultValue: '192.168.10.11;192.168.10.12', description: 'Client IPs')
    }

    environment {
        INI_FILE = 'inventory.ini'
    }

    stages {
        stage('Generate inventory') {
            steps {
                script {
                    def serverList = params.servers.split(';')
                    def clientList = params.clients.split(';')

                    def inventoryContent = new StringBuilder()
                    inventoryContent << "[servers]\n"
                    serverList.each { ip ->
                        inventoryContent << "${ip} ansible_user=vagrant ansible_password=vagrant\n"
                    }

                    inventoryContent << "\n[clients]\n"
                    clientList.each { ip ->
                        inventoryContent << "${ip} ansible_user=vagrant ansible_password=vagrant\n"
                    }

                    writeFile file: env.INI_FILE, text: inventoryContent.toString()
                }
            }
        }

        stage('Install consul') {
            steps {
                sh 'ansible-playbook -i inventory.ini playbook.yaml'
            }
        }

        stage('Start server') {
            steps {
                sh 'ansible-playbook -i inventory.ini consul_deploy.yaml'
            }
        }

        stage('Join clients') {
            steps {
                sh 'ansible-playbook -i inventory.ini consul_join.yaml'
            }
        }
    }
}