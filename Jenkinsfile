groovy
pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    tools {
        maven 'Maven-3.9.12'
    }

    environment {
        AWS_DEFAULT_REGION = 'eu-west-1'
    }

    stages {

        stage('Discover Application Servers') {
            steps {
                script {
                    env.APP1_IP = sh(
                        script: '''
                            aws ec2 describe-instances \
                              --filters "Name=tag:Name,Values=ubuntu-private-app1" \
                                        "Name=instance-state-name,Values=running" \
                              --query "Reservations[].Instances[].PrivateIpAddress" \
                              --output text \
                              --region eu-west-1
                        ''',
                        returnStdout: true
                    ).trim()

                    env.APP2_IP = sh(
                        script: '''
                            aws ec2 describe-instances \
                              --filters "Name=tag:Name,Values=amazon-linux-private-app2" \
                                        "Name=instance-state-name,Values=running" \
                              --query "Reservations[].Instances[].PrivateIpAddress" \
                              --output text \
                              --region eu-west-1
                        ''',
                        returnStdout: true
                    )

                    echo "Discovered App 1 IP: ${env.APP1_IP}"
                    echo "Discovered App 2 IP: ${env.APP2_IP}"

                    if (!env.APP1_IP || !env.APP2_IP) {
                        error "Could not discover one or both application servers."
                    }
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

        stage('Create ZIP') {
            steps {
                sh 'zip -j devops-project.zip index.html'
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
    }

    post {
        success {
            archiveArtifacts artifacts: 'devops-project.zip', fingerprint: true
        }
    }
}
