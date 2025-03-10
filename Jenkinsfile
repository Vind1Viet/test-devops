pipeline {
    agent any
    environment {
        CHANGED_FILES = ""
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    CHANGED_FILES = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim()
                    echo "Changed files:\n${CHANGED_FILES}"
                }
            }
        }

        stage('Determine Affected Service') {
            steps {
                script {
                    def services = ['vets-service', 'customers-service', 'visits-service', 'api-gateway', 'config-server', 'discovery-server']
                    def affectedServices = services.findAll { service ->
                        return CHANGED_FILES.split('\n').any { it.startsWith(service + '/') }
                    }
                    if (affectedServices.isEmpty()) {
                        echo "No relevant service changes, skipping build."
                        currentBuild.result = 'SUCCESS'
                        return
                    }
                    env.AFFECTED_SERVICES = affectedServices.join(' ')
                    echo "Affected services: ${env.AFFECTED_SERVICES}"
                }
            }
        }

        stage('Test and Build') {
            when {
                expression { env.AFFECTED_SERVICES != null && env.AFFECTED_SERVICES != "" }
            }
            steps {
                script {
                    env.AFFECTED_SERVICES.split().each { service ->
                        echo "Building and testing ${service}"
                        sh """
                        cd ${service}
                        ../../mvnw clean test
                        ../../mvnw package
                        """
                    }
                }
            }
        }
    }
}
