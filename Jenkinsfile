pipeline{
    agent any

    stages{
        stage("Checking node version in jenkins") {
            steps{
                sh '''
                    node -v
                    npm -v
                '''
        }
        }
    }
}