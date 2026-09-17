pipeline {

    agent any

    options {
        disableConcurrentBuilds()
        timeout(time: 20, unit: 'MINUTES')
    }

    tools {
        maven 'Maven-3.9.12'
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
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Create ZIP') {
            steps {
                sh '''
                    rm -f devops-project.zip
                    zip -j devops-project.zip index.html
                    ls -lh devops-project.zip
                '''
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
                        unzip -p devops-project.zip index.html > webserver1.html

                        scp -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            webserver1.html \
                            "$SSH_USER@$APP1_IP:/tmp/webserver1.html"

                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@$APP1_IP" \
                            'sudo mv /tmp/webserver1.html /var/www/html/webserver1.html && sudo chmod 644 /var/www/html/webserver1.html'

                        rm -f webserver1.html
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
                        unzip -p devops-project.zip index.html > webserver2.html

                        scp -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            webserver2.html \
                            "$SSH_USER@$APP2_IP:/tmp/webserver2.html"

                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            -i "$SSH_KEY" \
                            "$SSH_USER@$APP2_IP" \
                            'sudo mv /tmp/webserver2.html /usr/share/nginx/html/webserver2.html && sudo chmod 644 /usr/share/nginx/html/webserver2.html'

                        rm -f webserver2.html
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
                            'sudo test -f /var/www/html/webserver1.html'
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
                            'sudo test -f /usr/share/nginx/html/webserver2.html'
                    '''
                }

                echo 'Deployment verification successful on both application servers.'
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'devops-project.zip', fingerprint: true
        }
    }
}
