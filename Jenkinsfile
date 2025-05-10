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
                    npm insall --no-audit
                '''


        }
        }
    }
}