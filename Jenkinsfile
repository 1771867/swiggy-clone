pipeline{
     agent any
     
     tools{
         jdk 'jdk11'
         nodejs 'node16'
     }
     
     environment{
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub') // Credentials ID in Jenkins
        DOCKER_IMAGE = 'keshu1771867/myapp' // Docker image name
        DOCKER_TAG = 'latest'  // Docker image tag
        DOCKER_REGISTRY = 'keshu1771867' // Docker Hub username or registry URL
        AZURE_CREDENTIALS = credentials('jenkins') // Replace with your SP credential ID in Jenkins
        RESOURCE_GROUP = 'testrg' // Your AKS resource group name
        AKS_CLUSTER_NAME = 'testaks' // Your AKS cluster name
        AZURE_CREDENTIALS_TENANT = 'dcba8006-a1a1-4edc-ac4b-61c18092771d'
     }
     
     stages{
         
         stage('checkout from git'){
             steps{
                 checkout changelog: false, poll: false, scm: scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/1771867/swiggy-clone.git']])
             }
         }
         stage("Docker Build & Push"){
             steps{
                 script{
                        sh  """
                         docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                         echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin
                         docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                         docker push ${DOCKER_IMAGE}:${DOCKER_TAG} 
                        """
                     
                 }
             }
         }
         stage('Azure Login') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Login to Azure using the Service Principal
                    sh '''
                    az login --service-principal -u 9f04b434-4dbe-4610-abc1-43bb1fb4511e -p 9nG8Q~7t1wSuxnpGK.XSzbMVkVJBNQdRQ4eCRbyQ --tenant dcba8006-a1a1-4edc-ac4b-61c18092771d
                    '''
                }
            }
        }

        stage('Get AKS Credentials') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Get the AKS cluster credentials to configure kubectl
                    sh '''
                    az aks get-credentials --resource-group ${RESOURCE_GROUP} --name ${AKS_CLUSTER_NAME} --overwrite-existing
                    '''
                }
            }
        }

        stage('Deploy to Kubernets'){
            when {
                branch 'main'
            }
             steps{
                 script{
                     dir('Kubernetes') {
                         sh 'kubectl delete --all pods'
                         sh 'kubectl apply -f deployment.yml'
                         sh 'kubectl apply -f service.yml'
                            
                     }
                 }
             }
         }
         
     }
         
     
}
