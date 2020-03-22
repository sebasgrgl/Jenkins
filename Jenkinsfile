node{
//Define all variables
  def project = 'eighth-service-250517'
  def appName = 'hello-world'
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
  	 withCredentials([[$class: 'FileBinding', credentialsId:'Gcloud',variable:'Gcloud']]) {
      sh ("${GCLOUD_PATH}/gcloud auth activate-service-account --key-file ${Gcloud}")
      // push the docker image to google cloud container registry with proper image tag
      sh("${GCLOUD_PATH}/gcloud docker -- push ${imageTag}")
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
                   withEnv(['GCLOUD_PATH=/home/bontsrik/google-cloud-sdk/bin']){
                   // Create a kubernetes cluster in the google-cloud
                   sh("${GCLOUD_PATH}/gcloud container clusters create hello-world-cluster --num-nodes 1 --machine-type n1-standard-1 --zone us-central1-c --project=eighth-service-250517")
                   // Connect to the kuberntes cluster created in the previous command
                   sh("${GCLOUD_PATH}/gcloud container clusters get-credentials hello-world-cluster --zone us-central1-c --project eighth-service-250517")
                   //Deploy the hello-world application on to the kubernetes engine exposed on to the port 8080
                   sh("kubectl run hello-world --image=${imageTag} --port=8080")
                   //Open the engine to allow external traffic
                   sh("kubectl expose deployment hello-world --type=LoadBalancer")
                   // Adding the 60 sec delay so as to view the Internal and External Ip addresses. 
                   sh("sleep 60")
                   sh("kubectl get services")
                   }

                   break
         
 }
 }
}