pipeline {
    agent any
    environment {
        kubernetes_server_private_ip = credentials('kubernetes_server_private_ip')
    }

    stages {
        stage('gitclone') {
            steps {
                git url: "https://github.com/Gerardbulky/install-grafana-usx-ansible.git", branch: "main"
                echo "Workspace Path: ${WORKSPACE}/deployment.yaml"
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t bossmanjerry/vault-images:${env.BUILD_NUMBER} ."
            }
        }
        stage('Docker Push') {
            steps {
                //Logging into vault
                withVault(
                    configuration: [
                        disableChildPoliciesOverride: false, 
                        timeout: 60, 
                        vaultCredentialId: 'vault-jenkins-role', 
                        vaultUrl: 'http://51.20.76.240:8200'], 
                        vaultSecrets: [
                            [path: 'secrets/creds/my-secret-text', secretValues: [[vaultKey: 'username'], [vaultKey: 'password']]]]) {
                    sh 'docker login -u $username -p $password'
                }
                sh "docker push bossmanjerry/vault-images:${env.BUILD_NUMBER}"
            }
        }
        stage('Update Manifest') {
            steps {
                script {
                    // Retrieve GitHub credentials from Vault
                    def vaultResponse = withVault(
                        configuration: [
                            disableChildPoliciesOverride: false,
                            timeout: 60,
                            vaultCredentialId: 'vault-jenkins-role',
                            vaultUrl: 'http://51.20.76.240:8200'
                        ],
                        vaultSecrets: [
                            [path: 'secrets/creds/my-secret-text', secretValues: [[vaultKey: 'github_username'], [vaultKey: 'github_password']]]
                        ]
                    ) {
                        // Configure Git
                        sh "git config user.email gerardambe@yahoo.com"
                        sh "git config user.name Gerardbulky"
                        
                        // Modify deployment.yaml
                        sh "sed -i 's+bossmanjerry/vault-images.*+bossmanjerry/vault-images:${env.BUILD_NUMBER}+g' deployment.yaml"
                        
                        // Add, commit, and push changes to GitHub
                        sh "git add ."
                        sh "git commit -m 'Changed github manifest file: ${env.BUILD_NUMBER}'"
                        sh 'git push https://$github_password@github.com/$github_username/install-grafana-usx-ansible.git HEAD:main'
                    }
                }
            }
            
        }
        
    }
}
