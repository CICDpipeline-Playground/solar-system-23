pipeline{
    agent any

    tools {
        nodejs 'Nodejs-gb-23.8.0'
    }

    environment {
        MONGO_URI = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
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
                            --format \'ALL\'
                            --prettyPrint''', odcInstallation: 'owasp-12.1.1'
                        
                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true

                        junit allowEmptyResults: true, skipOldReports: true, stdioRetention: '', testResults: 'dependency-check-junit.xml'

                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])

                    }
                }
            }
        }

        stage ("Executing unit tests"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'mongo-db-gb', passwordVariable: 'MONGO_PASSWORD', usernameVariable: 'MONGO_USERNAME')]) {  
                     sh 'npm test'
            }
                junit allowEmptyResults: true, skipOldReports: true, stdioRetention: '', testResults: 'test-results.xml'     
            }
        }
    }
}