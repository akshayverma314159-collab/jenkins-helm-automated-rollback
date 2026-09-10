pipeline {
    agent any
    environment {
        RELEASE_NAME = 'sample-app'
        NAMESPACE = 'production'
    }
    stages {
        stage('Deploy via Helm') {
            steps {
                sh 'helm upgrade --install ${RELEASE_NAME} ./helm-chart --namespace ${NAMESPACE} --create-namespace'
            }
        }
        stage('Verify Health & Automated Rollback') {
            steps {
                script {
                    // Wait for pods to initialize
                    sleep time: 30, unit: 'SECONDS'
                    
                    // Simulate checking Prometheus for 5xx errors. If this returns an error code, the catch block triggers.
                    def healthCheck = sh(script: 'curl -s -f http://sample-app.production.svc.cluster.local/health || exit 1', returnStatus: true)
                    
                    if (healthCheck != 0) {
                        echo "Health check failed! Error rate spike detected. Initiating automated rollback."
                        sh 'helm rollback ${RELEASE_NAME} 0 --namespace ${NAMESPACE}'
                        error("Pipeline aborted due to health check failure. Successfully rolled back to previous state.")
                    } else {
                        echo "Deployment healthy. SLOs maintained."
                    }
                }
            }
        }
    }
}
