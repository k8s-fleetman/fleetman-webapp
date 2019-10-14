pipeline {
   agent any

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)
     
     SERVICE_NAME = "fleetman-webapp"
     REPOSITORY_TAG="${DOCKERHUB_URL}/${SERVICE_NAME}:${BUILD_ID}"
   }

   stages {
      stage('Preparation') {
         steps {
            cleanWs()
            git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }
      stage('Build') {
         steps {
            sh 'echo No build required for Webapp.'
         }
      }
      stage('SonarQube') {
         steps {
            sh '''sonar-scanner -Dsonar.projectKey=fleet-app -Dsonar.sources=. -Dsonar.host.url=http://sonarqube.eqslearning.com:9000 -Dsonar.login=25f9663ddadfd71eeaf3795ba07e4f0e0004f9b0'''
         }
      }
      stage('Build Image') {
         steps {
           sh 'scp -r ${WORKSPACE} jenkins@${DOCKER_HOST_IP}:/home/jenkins/docker/${BUILD_ID}'
           sh 'ssh jenkins@${DOCKER_HOST_IP} docker image build -t ${REPOSITORY_TAG} /home/jenkins/docker/${BUILD_ID}'
           sh 'ssh jenkins@${DOCKER_HOST_IP} docker image ls'
           sh 'ssh jenkins@${DOCKER_HOST_IP} rm -rf /home/jenkins/docker/${BUILD_ID}'
         }
      }
      stage('Push Image to repo') {
          steps {
           sh 'ssh jenkins@${DOCKER_HOST_IP} docker push ${REPOSITORY_TAG}'
          }
      }
      stage('Deploy to Cluster') {
          steps {
            //sh 'envsubst < ${WORKSPACE}/deploy.yaml | kubectl apply -f -'
            sh 'cat deploy.yaml'
            sh """sed -i 's+REPOSITORY_TAG+'"${REPOSITORY_TAG}"'+' deploy.yaml"""
            sh 'cat deploy.yaml'
            sh 'kubectl apply -f deploy.yaml'
          }
      }
   }
}
