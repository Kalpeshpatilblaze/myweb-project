pipeline {

    agent any

    stages {

        stage('Git Checkout') {
            steps {
                git 'https://github.com/Kalpeshpatilblaze/myweb-project.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Image Build') {
            steps {
                sh 'docker build . -t myimage:$BUILD_NUMBER'
            }
        }

        stage('Docker Image Tag and Push') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub_id',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker tag \
                            myimage:$BUILD_NUMBER \
                            $DOCKER_USERNAME/myimage:$BUILD_NUMBER

                        docker push \
                            $DOCKER_USERNAME/myimage:$BUILD_NUMBER

                        docker logout
                    '''
                }
            }
        }

        stage('Update Kubernetes Image') {
            steps {

                sh '''
                    sed -i "s|image:.*|image: kalpuaggressive/myimage:$BUILD_NUMBER|" deployments.yml

                    echo "===== Kubernetes Image ====="
                    grep "image:" deployments.yml
                '''
            }
        }

        stage('Kubernetes Deployment') {
            steps {

                sh '''
                    kubectl apply -f deployments.yml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {

                sh '''
                    echo "===== Deployment ====="
                    kubectl get deployment mywebdeployment

                    echo "===== Rollout Status ====="
                    kubectl rollout status \
                        deployment/mywebdeployment \
                        --timeout=5m

                    echo "===== Pods ====="
                    kubectl get pods -l app=myweb

                    echo "===== Service ====="
                    kubectl get service mywebservice
                '''
            }
        }
    }

    post {

        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }

        always {
            cleanWs()
        }
    }
}
