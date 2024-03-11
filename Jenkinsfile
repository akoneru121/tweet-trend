pipeline {
    agent {
        node{
            label 'maven'
        }
    }
     stages {
        stage('Approval') {
            steps {
                script {
                    // Pause pipeline and wait for manual approval
                    def userInput = input(
                        id: 'userInput',
                        message: 'Do you want to proceed with the deployment?',
                        parameters: [choice(name: 'Proceed', choices: 'Yes', description: 'Proceed with deployment')]
                    )

                    // Check user input
                    if (userInput == 'Yes') {
                        echo 'Proceeding with deployment...'
                    } else {
                        error('Deployment aborted by user.')
                    }
                }
            }
        }
    
        stage('SonarQube analysis') {
        environment {
        scannerHome = tool 'akoneru-sonar-scanner'
        }
        steps{
        withSonarQubeEnv('akoneru-sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
          sh "${scannerHome}/bin/sonar-scanner"
        }
    }
  }
}
}
