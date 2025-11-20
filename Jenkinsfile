pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "aymen28500" // replace this with your docker-id
DOCKER_IMAGE = "jenkins-exam"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
    stage('Build & Push images') {
      environment
      {
        DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
      }    
      steps {
        sh '''
            docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG cast-service
            docker login -u $DOCKER_ID -p $DOCKER_PASS
            docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
        '''    
      }        
    }
    
    stage('Deploiement en DEV') {
      environment {
      KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
      }
      steps {  
        sh '''
        export KUBECONFIG=$KUBECONFIG
        cp charts/values.yaml values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        helm upgrade --install app charts --values=values.yml --namespace dev
        '''   
      }
    }

    stage('Deploiement en STAGING') {
      environment {
      KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
      }
      steps {
        sh '''
        rm -Rf .kube
        mkdir .kube
        cat $KUBECONFIG > .kube/config
        cp charts/values.yaml values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        helm upgrade --install app charts --values=values.yml --namespace staging
        '''
      }
    }

    stage('Deploiement en PROD') {
      environment {
      KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
      }
      steps {
        timeout(time: 15, unit: "MINUTES") {
          input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
          rm -Rf .kube
          mkdir .kube
          cat $KUBECONFIG > .kube/config
          cp charts/values.yaml values.yml
          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
          helm upgrade --install app charts --values=values.yml --namespace prod
          '''
        }
      }
    }
  }
}
