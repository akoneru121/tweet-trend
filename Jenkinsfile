def registry = 'https://astech05.jfrog.io'
//def imageName = 'akoneru122.jfrog.io/docker-trial/ttrend'
def version   = '2.1.4'
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


    stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"1e0a6514-a1aa-4d09-a361-005f87be641f"
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

    stage(" Docker Build ") {
      steps {
        script {
           echo '<--------------- Docker Build Started --------------->'
           app = docker.build(imageName+":"+version)
           echo '<--------------- Docker Build Ends --------------->'
        }
      }
    }  

    stage (" Docker Publish "){
        steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'  
                docker.withRegistry(registry, '1e0a6514-a1aa-4d09-a361-005f87be641f'){
                    app.push()
                }    
               echo '<--------------- Docker Publish Ended --------------->'  
            }
        }
    }

}
}

