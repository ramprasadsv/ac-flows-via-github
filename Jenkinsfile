import groovy.json.JsonSlurper
import groovy.json.JsonOutput; 

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurper().parseText(json)
}
def toJSON(def json) {
    new groovy.json.JsonOutput().toJson(json)
}

String INSTANCEARN = ""
String TRAGETINSTANCEARN = ""
String TARGETFLOWID = ""
String TARGETJSON = ""

pipeline {
    agent any
    stages {
      
      
        stage('git repo & clean') {
            steps {
              script{
                    try{
                      sh(script: "rm -r ac-flows-via-github", returnStdout: true)
                    }catch(Exception e){
                      println ("Exception occured " + e.toString())
                    }
                    sh(script: "git clone https://github.com/ramprasadsv/ac-flows-via-github.git", returnStdout: true)
                    sh(script: "ls -ltr", returnStatus: true)
              }
            }
        }
      
      
        stage('read flow from git') {
            steps{
                echo 'Reading the contact flow content '
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def data = sh(script: 'cat contactflow.json', returnStdout: true).trim()    
                        echo data
                        def data2 = sh(script: 'cat arnmapping.json', returnStdout: true).trim()    
                        echo data2
                        def data3 = sh(script: 'cat parameters.json', returnStdout: true).trim()    
                        echo data3
                      
                        def flow = jsonParse(data)
                        def arnmapping = jsonParse(data2)
                        def instanceMapping = jsonParse(data3)
                        INSTANCEARN = instanceMapping.primaryInstance
                        TRAGETINSTANCEARN = instanceMapping.targetInstance
                        TARGETFLOWID = instanceMapping.targetFlowId
                        String content = flow.ContactFlow.Content    
                        echo content
                      
                        for(i = 0; i < arnmapping.size(); i++){
                            echo "Checking on ARN : ${arnmapping[i].sourceARN}"
                            println(content.indexOf(arnmapping[i].sourceARN, 1))
                            content = content.replaceAll(arnmapping[i].sourceARN, arnmapping[i].targetARN)
                        }
                        echo content                        
                      
                        String json = toJSON(content)
                        echo json.toString()
                        TARGETJSON = json.toString()                        
                    }
                }
            }
        }
      
      
        stage('deploy flow after reading from git') {
            steps {
                echo "updating flow content after reading from git "
                withAWS(credentials: '71b568ab-3ca8-4178-b03f-c112f0fd5030', region: 'us-east-1') {
                    script {
                        def di =  sh(script: "aws connect update-contact-flow-content --instance-id ${TRAGETINSTANCEARN} --contact-flow-id ${TARGETFLOWID} --content ${TARGETJSON}", returnStdout: true).trim()
                        echo di
                    }
                }
            }
        }

        stage('test the flow using Cyara') {
            steps{
                  //sleep(time:60,unit:"SECONDS")
                  //build job: 'Cyara Test Job', propagate: true, wait: true
            }
        }
        
        
    }
}

