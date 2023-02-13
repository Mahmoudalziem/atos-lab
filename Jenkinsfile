

pipeline{
    agent any
    environment{
        NAME="Mahmoud Abd Alziem"
        SCANNER_HOME=tool 'sonarQube'
        ORGANIZATION="microservices"
        PROJECT_NAME="atos"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "34.123.134.84:32000"
        NEXUS_REPOSITORY = "atos-lab"
        NEXUS_CREDENTIAL_ID = "admin"
    }
     options {
        buildDiscarder logRotator( 
                    daysToKeepStr: '16', 
                    numToKeepStr: '10'
            )
    }

    stages{
        stage('Testing Source Code'){
            steps{
                withSonarQubeEnv(installationName: 'sonarQube',credentialsId: 'sonarQube') {
                    sh ''' 
                          $SCANNER_HOME/bin/sonar-scanner 
                    '''
                }               
            }
        }
        

        stage('Build Artifact'){
            steps{
                    sh ''' 
                        ./mvnw package
                         java -jar target/*.jar
                         ./mvnw spring-boot:build-image
                    '''
            }
        }

       
        stage('Push Image'){
            steps{
                catchError(message : "Message") {
                     script{
                        docker.withRegistry('34.123.134.84:32000', 'nexus-credentials') {
                            app.push("${env.BUILD_NUMBER}")
                            app.push("latest")
                        }
                }
                }
            }
        }

        stage("Run Docker image"){
            steps{
                catchError(message : "Message"){
                    sh '''
                        docker run -it -d -p 80:80 --name ecommerce azima/jenkins:${BUILD_NUMBER}
                        echo done
                    '''
                }
            }
        }
    }

    post{
        always{
            echo "Start Stages Pipeline"
        }
        success{
            slackSend color: "#fff", message: "Success Publish Ecommerce Application"
        }
        failure{
            slackSend color: "#000", message: "Failed Publish Ecommerce Application"
        }
    }
}
