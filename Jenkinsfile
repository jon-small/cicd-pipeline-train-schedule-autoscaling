pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "jonsmall333/train-schedule-autoscale"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                //Logic to deploy K8s pod to raspbpi cluster using kubernetes CLI plugin and K3s client
                withKubeConfig([credentialsId: 'kubeconfig-file']) {
                      sh 'k3s kubectl apply -f $JENKINS_HOME/workspace/train-schedule-autoscale_master/train-schedule-kube.yml'
                }
            }
        }
    }
}

/*pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "jonsmall333/train-schedule"
    }
    stages {   
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                //Logic to deploy K8s pod to raspbpi cluster using kubernetes-cd plugin v1.0.0
                withKubeConfig([credentialsId: 'kubeconfig-file']) {
                      sh 'k3s kubectl apply -f $JENKINS_HOME/workspace/train-schedule-kubernetes_master/train-schedule-kube.yml'
                }
            }
        }
    }
}*/
