#!/usr/bin/env groovy
import groovy.json.JsonSlurper
import groovy.json.JsonBuilder

node {

    def server_url = "http://jfrog.local/artifactory"
    def distribution_url = "http://jfrog.local:8083/api/v1/"
    def latestBuildNumber
    def dockerManifestChecksum
    def dockerImage
    def sourceArtifactoryId

  
    
    stage ('Get Source Artifactory Id') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: CREDENTIALS, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
            def getServiceIdCommand = ["curl", "-s", "-u$USERNAME:$PASSWORD", "$server_url/api/system/service_id"]
            sourceArtifactoryId = getServiceIdCommand.execute().text
        }
    }

    stage ('Create Release Bundle') {
        createRBDN (sourceArtifactoryId, distribution_url)
    }
    
    stage ('Sign Release Bundle') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: CREDENTIALS, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
           def rbdnRequest = ["curl", "-X", "POST", "-H", "Content-Type: application/json", "-u", "$USERNAME:$PASSWORD", "${distribution_url}release_bundle/sol1/1.1/sign"]
    
           try {
              def rbdnResponse = rbdnRequest.execute().text
              println "Release Bundle Sign Response is: " + rbdnResponse
           } catch (Exception e) {
              println "Caught exception trying to sign release bundle. Message ${e.message}"
              throw e
           }
        }
    }
    
    stage ('Distribute') {
        
     def distributePayload = """ {
      "dry_run":"false",
      "distribution_rules": [
        {
            "service_name": "edge*"
        }
      ]
      }"""

   withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: CREDENTIALS, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
       def rbdnRequest = ["curl", "-X", "POST", "-H", "Content-Type:application/json", "-d", "${distributePayload}", "-u", "$USERNAME:$PASSWORD", "${distribution_url}distribution/sol1/1.1"]

       try {
          def rbdnResponse = rbdnRequest.execute().text
          println "Distribution Response is: " + rbdnResponse
       } catch (Exception e) {
          println "Caught exception when requesting distribution. Message ${e.message}"
          throw e
       }
   }   
        
    }    
        
        
}

def createRBDN (sourceArtifactoryId, distribution_url) {
   def aqlDockerString = 'items.find({\\"repo\\":\\"docker-prod-local\\",\\"path\\":{\\"$match\\":\\"*docker-app/latest*\\"}})'
  
   def aqlUbuntuString = 'items.find({\\"repo\\":\\"docker-remote-cache\\",\\"path\\":{\\"$match\\":\\"*ubuntu/latest*\\"}})'
  
   def releaseBundle = """ {
      "name":"sol1",
      "version": "1.1",
      "description":"solution1",
      "dry_run":"false",
      "spec": {
            "source_artifactory_id": "$sourceArtifactoryId",
            "queries":[
                {
                "aql": "${aqlDockerString}"
                },
                {
                "aql": "${aqlUbuntuString}"
                }
            ]
      }
   }"""
   println releaseBundle
   withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: CREDENTIALS, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
       def rbdnRequest = ["curl", "-X", "POST", "-H", "Content-Type: application/json", "-d", "${releaseBundle}", "-u", "$USERNAME:$PASSWORD", "${distribution_url}release_bundle"]

       try {
          def rbdnResponse = rbdnRequest.execute().text
          println "Release Bundle Response is: " + rbdnResponse
       } catch (Exception e) {
          println "Caught exception. Message ${e.message}"
          throw e
       }
    }

}


