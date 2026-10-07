pipeline {
    agent any

    environment {
        OPENSHIFT_API = 'https://172.22.80.1:6443' // Your local CRC IP
        NAMESPACE     = 'default'
        APP_NAME      = 'httpd-stuck-app'
    }

    stages {
        stage('Login to OpenShift') {
            steps {
                withCredentials([string(credentialsId: 'openshift-token', variable: 'OPENSHIFT_TOKEN')]) {
                    sh 'oc login ${OPENSHIFT_API} --token="${OPENSHIFT_TOKEN}" --insecure-skip-tls-verify=true'
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

        stage('Simulate & Verify App Degradation') {
            steps {
                script {
                    echo "Waiting 15 seconds for application to enter degraded state..."
                    sleep 15

                    def podName = sh(
                        script: "oc get pods -l app=${APP_NAME} -n ${NAMESPACE} -o jsonpath='{.items[0].metadata.name}'",
                        returnStdout: true
                    ).trim()

                    echo "Testing HTTP health endpoint on pod: ${podName}"
                    
                    // Run curl inside the container to fetch the HTTP status code on /health
                    def httpCode = sh(
                        script: "oc exec ${podName} -n ${NAMESPACE} -- curl -s -o /dev/null -w '%{http_code}' http://localhost/health",
                        returnStdout: true
                    ).trim()

                    echo "HTTP Response Code from /health: ${httpCode}"

                    if (httpCode == "500") {
                        echo "--------------------------------------------------------"
                        echo "SUCCESS: Pod is running, but /health is returning HTTP 500!"
                        echo "Agentic app can now detect HTTP 500 and restart this pod."
                        echo "--------------------------------------------------------"
                    } else {
                        error("Expected HTTP 500 but got ${httpCode}")
                    }
                }
            }
        }
    }
}
