pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('my-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('my-aws-secret-access-key')
        AWS_DEFAULT_REGION = 'us-east-1'
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout code from Git repository
                git 'https://github.com/your/repository.git'
            }
        }

         stage('Infrastructure Provisioning') {
            steps {
                // Execute Terraform or CloudFormation scripts to provision infrastructure
                sh 'terraform init'
                sh 'terraform plan -out=tfplan'
                sh 'terraform apply -auto-approve tfplan'
            }
        }

        // Get EC2 IPs from autoscaling group for Ansible host file
       

         stage('Retrieve servers') {
            
            steps {
            
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-key', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) {
               
                script {
                     def autoscalingGroupName = 'haautoscaling'

                    // Retrieve instance IDs from the autoscaling group
                    def instanceIds = sh(script: "aws autoscaling describe-auto-scaling-instances --query 'AutoScalingInstances[?AutoScalingGroupName==`$autoscalingGroupName`].InstanceId' --output text", returnStdout: true).trim().split()
                    
                     
                    // Retrieve private IPs of instances
                    def privateIps = []
                    for (instanceId in instanceIds) {
                        def ip = sh(script: "aws ec2 describe-instances --instance-ids ${instanceId} --query 'Reservations[0].Instances[0].PrivateIpAddress' --output text", returnStdout: true).trim()                        
                        privateIps.add(ip)
                    }
                    
                    def joinedIps = privateIps.join('\n')
                    writeFile file: "hosts", text: "[autoscaling_group]\n${privateIps.join('\n')}\n"   
             }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                script {
                    // Run Ansible playbook using the generated hosts file
                    echo "calling Anible playbooks to configure ec2 instances"
                    cd ansible
                    sh "ansible-playbook -i hosts configure-ec2-servers.yml"
                }
            }
        }
    
        
        
        
        
        
       
        
        
        
        

        
    }
}
