pipeline {
   agent any
   parameters {
        string(name:'region', defaultValue: 'us-east-2', description: 'AWS EKS CLUSTER REGION ')
        string(name: 'instance_role', defaultValue: 'arn:aws:iam::719899497748:role/eks-demo-worker-NodeInstanceRole-PEM2N4JO09Y1', description: 'Instance Role Arn')
        string(name: 'eks_cluster_name', defaultValue: 'eks_demo', description:'EKS Cluster Name')

   }
   stages {
      stage('Start Deploy APP On EKS Cluser') {
         steps {
            echo 'Start Deploy APP On EKS Cluser'
            print params.region
            print params.eks_cluster_name

         }
      }
       /*stage("Install kubectl") {
         steps {
             sh '''
                curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
                chmod +x ./kubectl
                mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
                echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
                kubectl version --short --client '''
         }
      } */
      /*stage("Install aws-iam-authenticator") {
         steps {
             withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                                  credentialsId    : 'eks_demo',
                                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){
                script{
                def region = "${params.region}"
                sh '''
                export AWS_REGION=${region}
                curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/aws-iam-authenticator
                chmod +x ./aws-iam-authenticator
                mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$HOME/bin:$PATH
                echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
                aws-iam-authenticator help
                printenv  '''
                }
             }
         }
      } */
      stage("Update Kube Config") {
          steps {
              withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                                  credentialsId    : 'eks_demo',
                                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){
                script{
                def cluster_name = params.eks_cluster_name
                print cluster_name
                sh """ aws eks --region ${params.region} update-kubeconfig --name ${cluster_name} """
                }
              }
          }
      }
      stage('Checkout EKS DEMO Project') {
          steps {
               git branch: 'master',
                credentialsId: 'madhuri_git',
                url: 'https://github.com/madhurikadam/eks_demo.git'
               sh "ls -lat"
               sh "pwd"
          }
      }
      stage('Set EKS Cluser Config') {
          steps {
              withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                                  credentialsId    : 'eks_demo',
                                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){
                script{
                def instance_role = "${params.instance_role}"
                sh '''
                   export INSTANCE_ROLE=${instance_role}
                   cp aws-auth-cm.yaml ~/.kube/aws-auth-cm.yaml
                   kubectl apply -f ~/.kube/aws-auth-cm.yaml
                   kubectl get svc  '''
                }
              }
          }
      }
      stage('Deploy Demo APP On EKS Cluser') {
          steps {
               withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                                  credentialsId    : 'eks_demo',
                                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]){
                 sh '''
                    kubectl apply -f helloworld.yaml
                    kubectl apply -f helloworld-service.yaml
                    kubectl get svc
                    sleep 20
                    kubectl get svc service-helloworld -o yaml '''
               }
          }
      }
   }
}
