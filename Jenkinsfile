pipeline {
    agent any

    environment {
        OPENSHIFT_API = 'https://api.crc.testing:6443'
        NAMESPACE     = 'customer-dev'
        APP_NAME      = 'httpd-stuck-app'
    }

    stages {
        stage('Login to OpenShift') {
            steps {
                withCredentials([string(credentialsId: 'openshift-token', variable: 'OPENSHIFT_TOKEN')]) {
                    bat 'oc login %OPENSHIFT_API% --token=%OPENSHIFT_TOKEN% --insecure-skip-tls-verify=true'
                }
            }
        }

        stage('Deploy App') {
            steps {
                script {
                    echo "Deploying HTTPD application to OpenShift namespace: ${env.NAMESPACE}..."
                    bat "oc apply -f httpd-stuck-app.yaml -n %NAMESPACE%"
        
                    echo "Waiting for deployment rollout to finish..."
                    // Increased timeout to 120s
                    bat "oc rollout status deployment/%APP_NAME% -n %NAMESPACE% --timeout=120s"
                }
              }
        }

        stage('Simulate & Verify App Degradation') {
            steps {
                script {
                    echo "Waiting 12 seconds for htaccess 500 trigger script to run..."
                    sleep 12
        
                    def podName = bat(
                        script: "oc get pods -l app=%APP_NAME% -n %NAMESPACE% -o jsonpath=\"{.items[0].metadata.name}\"",
                        returnStdout: true
                    ).trim()

                    podName = podName.tokenize('\r\n').last().trim()
        
                    echo "Testing HTTP health status on pod: ${podName}"
                    
                    def healthContent = bat(
                        script: "oc exec ${podName} -n %NAMESPACE% -- cat /usr/local/apache2/htdocs/health",
                        returnStdout: true
                    ).trim()
        
                    echo "Health File Content: ${healthContent}"
        
                    def htaccessContent = bat(
                        script: "oc exec ${podName} -n %NAMESPACE% -- cat /usr/local/apache2/htdocs/.htaccess",
                        returnStdout: true
                    ).trim()
        
                    echo ".htaccess Rules Active: ${htaccessContent}"
        
                    if (htaccessContent.contains("Redirect 500") || htaccessContent.contains("RewriteRule")) {
                        echo "--------------------------------------------------------"
                        echo "SUCCESS: Pod is active and 500 rule is live!"
                        echo "--------------------------------------------------------"
                    }
                }
            }
        }
    }
}
