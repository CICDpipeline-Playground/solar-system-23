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
                        dependencyCheck additionalArguments: '''--scan \\\'./\\\'
                            --out \\\'./\\\'
                            --format \\\'ALL\\\'
                            --prettyprint''', odcInstallation: 'owasp-12.1.1'
                    }
                }
            }
        }
    }
}