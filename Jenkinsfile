pipeline {
    agent any 
    environment {
        VERSION = "${env.BUILD_ID}"
    }
    stages {
        stage("sonar quality check") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }
                    
                    timeout(time: 1, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        } // Missing closing curly brace for "sonar quality check" stage

        stage("docker build & docker push") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'doc_pass', variable: 'doc_pass')]) {
                        sh '''
                            docker build -t 52.87.170.237:8083/springapp:${VERSION} .
                            docker login -u admin --password-stdin $doc_pass 52.87.170.237:8083 
                            docker push 52.87.170.237:8083/springapp:${VERSION}
                            docker rmi 52.87.170.237:8083/springapp:${VERSION}
                        '''
                    }
                }
            }
        }
    }
}
