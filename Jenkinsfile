            pipeline{
                agent any

                stages{
                    stage("Checking node version in jenkins") {
                        step(
                            sh '''
                                node -v
                                npm -v
                            '''
                        )
                    }
                }
            }