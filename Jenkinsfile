pipeline {
    agent any
    
    stages {       
       stage ("Build") {
           steps {
                sh "mvn clean install -f MyPhoenixApp/pom.xml"   
           }
       }
       
       stage ("code scan") {
           steps {
               withSonarQubeEnv("SonarQube") {
                sh "mvn sonar:sonar -f MyPhoenixApp/pom.xml"   
               }
           }
       }
       
       stage ("coverage") {
           steps {
               jacoco()
           }
       }
       
       stage ("Binary upload") {
           steps {
                nexusArtifactUploader artifacts: [[artifactId: 'MyPhoenixApp', classifier: '', file: 'MyPhoenixApp/target/MyPhoenixApp.war', type: 'war']], credentialsId: '50d7f8ac-08e5-4205-b2fb-7b8bba0233dc', groupId: 'com.gcp', nexusUrl: 'ec2-3-91-60-9.compute-1.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'

           }
       }
       
       stage ("DEV deploy") {
           steps {
                deploy adapters: [tomcat9(credentialsId: '8a9bb117-a190-4ac3-ba88-67c07c629477', path: '', url: 'http://ec2-35-153-52-5.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
           }
           
       }
    
        stage ("Slack notify") {
            steps {
                slackSend channel: 'my-sep-2023-weekday-batch', message: 'DEV deployment was done, please start testing in DEV env'
            }
        }
        
        //CD starts here..
        
    stage ("DEV approve") {
        steps {
            echo "Taking approval from DEV Manager for QA Deployment"     
            timeout(time: 3, unit: 'DAYS') {
            input message: 'Do you approve QA Deployment?', submitter: 'admin'
        }
      }
    }
    
    stage ("QA deploy") {
        steps {
            deploy adapters: [tomcat9(credentialsId: '8a9bb117-a190-4ac3-ba88-67c07c629477', path: '', url: 'http://ec2-35-153-52-5.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
      }
    }
    
    stage ("QA notify") {
        steps {
            slackSend channel: 'my-sep-2023-weekday-batch,qa-testing-team', message: 'QA Team - QA Deployment is done, please start functional testing'
      }
    }

    stage ("QA approve") {
        steps {
        echo "Taking approval from QA Manager for PROD Deployment"     
        timeout(time: 3, unit: 'DAYS') {
        input message: 'Do you approve PROD Deployment?', submitter: 'admin'
        }
     }
    }
    
    stage ("prod deploy") {
        steps {
                deploy adapters: [tomcat9(credentialsId: '8a9bb117-a190-4ac3-ba88-67c07c629477', path: '', url: 'http://ec2-35-153-52-5.compute-1.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
      }
    }
    
    stage ("PO notify") {
         steps {
         slackSend channel: 'qa-testing-team,product-owners-teams', message: 'Business users - prod deployment is done, please validte and inform end customers'
        }
      }
        
    }
}
