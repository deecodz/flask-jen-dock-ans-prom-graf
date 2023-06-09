pipeline {
    agent any
    
    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"
        DOCKER_HUB_USERNAME = ""
        DOCKER_HUB_PASSWORD = ""        
    }

    stages {
        stage('Build Docker image') {
            steps {
                script {
                    def app = docker.build("derao/flask-app:${env.BUILD_NUMBER}")
                    app.inside {
                        sh 'echo "Docker build complete"'
                    }
                }
            }
        }

        stage('Retrieve Docker Hub credentials') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    script {
                        env.DOCKER_HUB_USERNAME = "${DOCKER_HUB_USERNAME}"
                        sh 'echo $DOCKER_HUB_PASSWORD | docker login --username derao --password-stdin'
                    }
                }
            }
        }


        stage('Push Docker image to Registry') {
            steps {
                script {
                    try {
                        docker.withRegistry("", 'docker_credentials') {
                            def app = docker.image("derao/flask-app")
                            app.push("${env.BUILD_NUMBER}")
//                             app.push('latest')
                        }
                    } catch (Exception e) {
                        error("Docker push failed: ${e.getMessage()}")
                    }
                }
            }
        }
              
        stage('Deploy to Remote') {
            steps {
                echo 'Running Playbook'
                ansiblePlaybook become: true, credentialsId: 'github', inventory: 'host.ini', playbook: 'flask-dep.yml'
            }
        }
    }
}

