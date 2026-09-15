pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timeout(time: 20, unit: 'MINUTES')
    }

    tools {
        maven 'Maven-3.9.12'
    }

    environment {
        AWS_DEFAULT_REGION = 'eu-west-1'
        AWS_PAGER = ''
    }

    stages {

        stage('Set Application Servers') {
            steps {
                script {
                    env.APP1_IP = '10.0.3.226'
                    env.APP2_IP = '10.0.4.68'

                    echo "App 1 IP: ${env.APP1_IP}"
                    echo "App 2 IP: ${env.APP2_IP}"
                }
            }
        }

        stage('Checkout') {
            steps {
                echo 'Code checked out from GitHub'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Verify Package') {
            steps {
                sh '''
                    echo "Build artifacts:"
                    ls -lh target/

                    echo "Package contents:"
                    unzip -l target/devops-project-1.0.zip
                '''
            }
        }

        stage('Archive Package') {
            steps {
                archiveArtifacts artifacts: 'target/devops-project-1.0.zip', fingerprint: true
            }
        }

        stage('Deploy App 1') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'app1-ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        scp -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            target/devops-project-1.0.zip \
                            "$SSH_USER@$APP1_IP:/tmp/devops-project-1.0.zip"

                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@$APP1_IP" \
                            'sudo rm -rf /var/www/html/application && \
                             sudo mkdir -p /var/www/html/application && \
                             sudo unzip -o /tmp/devops-project-1.0.zip -d /var/www/html && \
                             sudo chmod -R 755 /var/www/html/application && \
                             sudo rm -f /tmp/devops-project-1.0.zip'
                    '''
                }
            }
        }

        stage('Deploy App 2') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'app2-amazonlinux-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        scp -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            target/devops-project-1.0.zip \
                            "$SSH_USER@$APP2_IP:/tmp/devops-project-1.0.zip"

                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@$APP2_IP" \
                            'sudo rm -rf /usr/share/nginx/html/application && \
                             sudo mkdir -p /usr/share/nginx/html/application && \
                             sudo unzip -o /tmp/devops-project-1.0.zip -d /usr/share/nginx/html && \
                             sudo chmod -R 755 /usr/share/nginx/html/application && \
                             sudo rm -f /tmp/devops-project-1.0.zip'
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {

                withCredentials([sshUserPrivateKey(
                    credentialsId: 'app1-ubuntu-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@$APP1_IP" \
                            'sudo test -f /var/www/html/application/index.html'
                    '''
                }

                withCredentials([sshUserPrivateKey(
                    credentialsId: 'app2-amazonlinux-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@$APP2_IP" \
                            'sudo test -f /usr/share/nginx/html/application/index.html'
                    '''
                }

                echo 'Deployment verification successful on both application servers.'
            }
        }
    }
}
