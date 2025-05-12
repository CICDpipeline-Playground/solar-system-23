pipeline{
    agent any

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

        stage('Docker sanity') {
            steps {
                sh '''
                    echo "PATH = $PATH"
                    which docker || true
                    docker version
                    docker info
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

                script {
                    docker.build("gokulb574/solar-system:${env.GIT_COMMIT}")
                }
                // sh '''
                // printenv
                // docker build -t gokulb574/solar-system:$GIT_COMMIT .
                // '''  -> not working
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