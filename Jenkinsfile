def registry = 'https://akoneru122.jfrog.io'
pipeline {
    agent {
        node{
            label 'maven'
        }
    }

        environment {
        PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
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
        
        stage("build"){
            steps {
                 echo "----------- build started ----------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                 echo "----------- build complted ----------"
            }
        }
        // stage("test"){
        //     steps{
        //         echo "----------- unit test started ----------"
        //         sh 'mvn surefire-report:report'
        //          echo "----------- unit test Complted ----------"
        //     }
        // }

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
   stage("Quality Gate"){
    steps {
        script {
        timeout(time: 1, unit: 'SECONDS') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
  }
        }
    }
}


         stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifact-cred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "maven-local-dem00/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }   

}
}

