pipeline {
    agent any

    environment {
        PI_HOST = 'rosenpi.local'
        PI_USER = 'krosengren'
        CHART_PATH = 'chart'
        ENV_VALUES = 'environments/rosenpi/values.yaml'
        RELEASE_NAME = 'byteboard-service'
        NAMESPACE = 'default'
    }

    stages {
        stage('Deploy to K3s') {
            steps {
                echo 'Deploying byteboard-service to K3s with Helm...'
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'rosenpi-ssh',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh '''
                        # Copy chart and environment values to Pi
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no -r ${CHART_PATH} ${PI_USER}@${PI_HOST}:~/byteboard-service-chart
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no ${ENV_VALUES} ${PI_USER}@${PI_HOST}:~/byteboard-service-chart/env-values.yaml

                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${PI_USER}@${PI_HOST} << 'ENDSSH'
                            cd ~/byteboard-service-chart
                            helm upgrade --install byteboard-service . \
                                -f values.yaml \
                                -f env-values.yaml \
                                --namespace default \
                                --wait \
                                --timeout 120s
                            kubectl get pods -l app=byteboard-service
                        ENDSSH
                    '''
                }
            }
        }
    }

    post {
        success { echo 'Helm deployment successful!' }
        failure { echo 'Helm deployment failed!' }
    }
}
