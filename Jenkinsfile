node{
//Define all variables
  def project = 'my-project'
  def appName = 'my-first-microservice'
  def serviceName = "${appName}-backend"  
  def imageVersion = 'development'
  def namespace = 'development'
  def imageTag = "gcr.io/${project}/${appName}:${imageVersion}.${env.BUILD_NUMBER}"
  
  //Checkout Code from Git
     checkout scm  

  //Stage 1 : Build the docker imagetag.
  stage('Build image') {
      // def customImage = docker.build("my-image:${imageTag}")
	  def dockerhome = tool 'docker'
	  sh("docker build -t ${imageTag} .")
  }
  
  //Stage 2 : Push the image to docker registry
  stage('Push image to registry') {
  	 withEnv(['GCLOUD_PATH=/home/bontsrik/google-cloud-sdk/bin']) {
  	 //withCredentials([file(credentialsId: 'Gcloud', variable: 'Gcloud')]) {
  	 withCredentials([[$class: 'FileBinding', credentialsId:'Gcloud1',variable:'Gcloud1']]) {
      sh ("${GCLOUD_PATH}/gcloud auth activate-service-account --key-file ${Gcloud1}")
     // sh("${GCLOUD_PATH}/gcloud auth print-access-token | docker login -u oauth2accesstoken --password-stdin https://gcr.io")
     //sh("${GCLOUD_PATH}/gcloud config set project my-project")
      sh("docker push ${imageTag}")
	 //}	 	
	}
   }
  }
  
  //Stage 3 : Deploy Application
  stage('Deploy Application') {
       switch (namespace) {
              //Roll out to Dev Environment
              case "development":
                   // Create namespace if it doesn't exist
                   sh("kubectl get ns ${namespace} || kubectl create ns ${namespace}")
           //Update the imagetag to the latest version
                   sh("sed -i.bak 's#gcr.io/${project}/${appName}:${imageVersion}#${imageTag}#' ./k8s/development/*.yaml")
                   //Create or update resources
           sh("kubectl --namespace=${namespace} apply -f k8s/development/deployment.yaml")
                   sh("kubectl --namespace=${namespace} apply -f k8s/development/service.yaml")
           //Grab the external Ip address of the service
                   sh("echo http://`kubectl --namespace=${namespace} get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
                   break
           
        //Roll out to Dev Environment
              case "production":
                   // Create namespace if it doesn't exist
                   sh("kubectl get ns ${namespace} || kubectl create ns ${namespace}")
           //Update the imagetag to the latest version
                   sh("sed -i.bak 's#gcr.io/${project}/${appName}:${imageVersion}#${imageTag}#' ./k8s/production/*.yaml")
           //Create or update resources
                   sh("kubectl --namespace=${namespace} apply -f k8s/production/deployment.yaml")
                   sh("kubectl --namespace=${namespace} apply -f k8s/production/service.yaml")
           //Grab the external Ip address of the service
                   sh("echo http://`kubectl --namespace=${namespace} get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
                   break
       
              default:
                   sh("kubectl get ns ${namespace} || kubectl create ns ${namespace}")
                   sh("sed -i.bak 's#gcr.io/${project}/${appName}:${imageVersion}#${imageTag}#' ./k8s/development/*.yaml")
                   sh("kubectl --namespace=${namespace} apply -f k8s/development/deployment.yaml")
                   sh("kubectl --namespace=${namespace} apply -f k8s/development/service.yaml")
                   sh("echo http://`kubectl --namespace=${namespace} get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
                   break
 }
 }
}