pipeline{
    agent any

    tools {
        nodejs 'Nodejs-gb-23.8.0'
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
                stage ("NPM dep checking") {
                    steps{
                        sh '''
                            npm audit --audit-level=critical
                            echo $?
                        '''
                    }
                }
                stage ("dummy stage"){
                    steps{
                        sh 'echo $?'
                    }
                }
            }
        }
    }
}