pipeline{
    agent {
        label 'slave-eu-1'
    }

    tools {
        nodejs 'Nodejs-gb-23.8.0'
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
        //https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#for-secret-text-usernames-and-passwords-and-secret-files
        MONGO_DB_CREDS = credentials('mongo-db-gb')
        MONGO_USERNAME = credentials('mdb-gb-uname')
        MONGO_PASSWORD = credentials('mdb-gb-pwd')
        //DOCKER_HOME = tool ('docker-latest-gb')
    }


    stages{
        stage("Checking node version in jenkins and installing dependencies") {
            steps{
                sh '''
                    node -v
                    npm -v
                    npm install --no-audit
                '''
        }
        }

        

        stage ("Dependency Scanning"){
            parallel {
                stage ("NPM DEP CHECKING") {
                    steps{
                        sh '''
                            npm audit --audit-level=critical
                            echo $?
                        '''
                    }
                }
                stage ("OWASP DEP CHECKING"){
                    steps{
                        dependencyCheck additionalArguments: '''--scan \'./\'
                            --out \'./\'
                            --format \'XML\'
                            --disableYarnAudit \
                            --prettyPrint''', odcInstallation: 'owasp-12.1.1'
                        
                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true

                    }
                }
            }
        }

        stage ("Executing unit tests"){
            steps{
                catchError(buildResult: 'SUCCESS', message: 'Unknown error. This will be fixed in the next release.', stageResult: 'UNSTABLE') {
                    
                    sh '''
                        echo colon-separated - $MONGO_DB_CREDS
                        echo Mongo-Uname - $MONGO_USERNAME
                        echo Mongo-Pwd - $MONGO_PASSWORD
                    '''
                    
                    sh 'npm test'
            
                    
                }
            }
        }

        stage ("Code Coverage"){
            steps {
                catchError(buildResult: 'SUCCESS', message: 'Shhh! This can be fixed in the next release.', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                    
                }
            }
        }

        stage("SAST-SonarQube"){
            steps {
                echo "skipping this stage. if needed will configure later"
            }
        }
        
        stage("Building Docker Image"){
            steps {

                // script {
                //     docker.build("gokulb574/solar-system:${env.GIT_COMMIT}")
                // }
                // sh '''
                // printenv
                sh 'docker build -t gokulb12/solar-system:$GIT_COMMIT .'
                // '''  -> not working
            }
        
        }

        stage("Trivy Vulnerability Scanning"){
            steps {
                //for this step I have installed trivy cli using script  not using packg in jenkins slave
                sh '''
                    trivy image gokulb12/solar-system:$GIT_COMMIT \
                        --severity MEDIUM,HIGH \
                        --format json --output trivy-image-MED-HIGH-vul-report.json \
                        --quiet \
                        --exit-code 0
                '''
                sh '''
                    trivy image gokulb12/solar-system:$GIT_COMMIT \
                        --severity CRITICAL \
                        --format json --output trivy-image-CRITICAL-vul-report.json \
                        --quiet \
                        --exit-code 1
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-image-MED-HIGH-vul-report.json', fingerprint: true
                    archiveArtifacts artifacts: 'trivy-image-CRITICAL-vul-report.json', fingerprint: true
                }
            }
        }

        stage ("Pushing Docker Image") {
            steps{
                withDockerRegistry(credentialsId: 'gokulb12') {
                    sh 'docker push gokulb12/solar-system:$GIT_COMMIT'
                }
            }
        }
        
        
    }
    post {
        always {
            junit allowEmptyResults: true, skipOldReports: true, stdioRetention: '', testResults: 'dependency-check-junit.xml'
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            junit allowEmptyResults: true, skipOldReports: true, stdioRetention: '', testResults: 'test-results.xml'     
            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'coverage/lcov-report/', reportFiles: 'index.html', reportName: 'Code Coverage Report.html', reportTitles: '', useWrapperFileDirectly: true])
        }
    }

}