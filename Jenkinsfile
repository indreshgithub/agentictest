pipeline {
    agent any

    environment {
        // Update these two variables to match your environment
        OPENSHIFT_API = 'https://192.168.0.170:6443' 
        NAMESPACE     = 'default'
        APP_NAME      = 'httpd-stuck-app'
    }

    stages {
        stage('Login to OpenShift') {
            steps {
                withCredentials([string(credentialsId: 'openshift-token', variable: 'OPENSHIFT_TOKEN')]) {
                    // Log into OpenShift CLI using stored credentials
                    sh "oc login ${OPENSHIFT_API} --token=${OPENSHIFT_TOKEN} --insecure-skip-tls-verify=true"
                    sh "oc project ${NAMESPACE}"
                }
            }
        }

        stage('Deploy App') {
            steps {
                script {
                    echo "Deploying HTTPD application to OpenShift..."
                    sh "oc apply -f httpd-stuck-app.yaml -n ${NAMESPACE}"
                }
            }
        }

        stage('Simulate Failure') {
            steps {
                script {
                    echo "Waiting 20 seconds for simulated process exhaustion..."
                    sleep 20

                    def podName = sh(
                        script: "oc get pods -l app=${APP_NAME} -n ${NAMESPACE} -o jsonpath='{.items[0].metadata.name}'",
                        returnStdout: true
                    ).trim()

                    echo "Checking health status on pod: ${podName}"
                    sh "oc exec ${podName} -n ${NAMESPACE} -- cat /usr/local/apache2/htdocs/health"
                    echo "ALERT: Pod is now degraded! Ready for Agentic App restart."
                }
            }
        }
    }
}
