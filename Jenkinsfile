pipeline {
    agent any

    environment {
        OPENSHIFT_API = 'https://api.crc.testing:6443' // Your local CRC IP
        NAMESPACE     = 'default'
        APP_NAME      = 'httpd-stuck-app'
    }

    stages {
        stage('Login to OpenShift') {
            steps {
                withCredentials([string(credentialsId: 'openshift-token', variable: 'OPENSHIFT_TOKEN')]) {
                    bat 'oc login https://host.docker.internal:6443 --token=%OPENSHIFT_TOKEN% --insecure-skip-tls-verify=true'
                }
            }
        }

        stage('Deploy App') {
            steps {
                script {
                    echo "Deploying HTTPD application to OpenShift..."
                    bat "oc apply -f httpd-stuck-app.yaml -n %NAMESPACE%"
                }
            }
        }

        stage('Simulate & Verify App Degradation') {
            steps {
                script {
                    echo "Waiting 15 seconds for application to enter degraded state..."
                    sleep 15
        
                    def podName = bat(
                        script: "oc get pods -l app=%APP_NAME% -n %NAMESPACE% -o jsonpath=\"{.items[0].metadata.name}\"",
                        returnStdout: true
                    ).trim()

                    // bat returns the executed command line on Windows along with stdout; clean up extra lines
                    podName = podName.tokenize('\n').last().trim()
        
                    echo "Testing HTTP health status on pod: ${podName}"
                    
                    // Read content directly without requiring curl
                    def healthContent = bat(
                        script: "oc exec ${podName} -n %NAMESPACE% -- cat /usr/local/apache2/htdocs/health",
                        returnStdout: true
                    ).trim()
        
                    echo "Health File Content: ${healthContent}"
        
                    // Check if .htaccess redirect rule exists (Simulating 500 error)
                    def htaccessContent = bat(
                        script: "oc exec ${podName} -n %NAMESPACE% -- cat /usr/local/apache2/htdocs/.htaccess",
                        returnStdout: true
                    ).trim()
        
                    echo ".htaccess Rules Active: ${htaccessContent}"
        
                    if (htaccessContent.contains("Redirect 500")) {
                        echo "--------------------------------------------------------"
                        echo "SUCCESS: Pod is active and .htaccess 500 rule is live!"
                        echo "Agentic app can now detect failure and trigger restart."
                        echo "--------------------------------------------------------"
                    }
                }
            }
        }
    }
}
