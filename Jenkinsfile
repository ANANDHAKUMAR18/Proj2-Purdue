pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USERNAME = 'anandhakumarg'
        DOCKER_IMAGE_NAME = 'anandhsandy'
        DOCKER_IMAGE_TAG = 'latest'
        ANSIBLE_PLAYBOOK_PATH = 'ansible/deploy.yaml'
        DYNAMIC_PLAYBOOK_PATH = 'dynamic_playbook.yaml'
        K8S_DEPLOYMENT_FILE= 'kubernetes/deployment.yaml'
        
       
    }
    
    tools
    {
        maven "Maven"
    }
    stages {
        stage('Code') {
            steps {
                 git branch: 'main', url: 'https://github.com/ANANDHAKUMAR18/Proj2-Purdue.git'
            }
        }
        
        stage('Compile and Test') {
            steps {
                sh 'mvn clean compile test'
            }
        }
        stage('Package') {
            steps
            {
                sh 'mvn package'
            }
        }
        stage('Build Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'jenkins_sudo_password', variable: 'SUDO_PASSWORD')]) {
                   script {
                    sh " sudo docker build -t ${DOCKER_USERNAME}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                   }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'docker_password', variable: 'DOCKER_PASSWORD'),
                string(credentialsId: 'jenkins_sudo_password',variable: 'SUDO_PASSWORD')
                ]) {
                    script {
                        sh "sudo docker login -u ${DOCKER_USERNAME} -p ${env.DOCKER_PASSWORD} ${DOCKER_REGISTRY}"
                        sh " sudo  docker push ${DOCKER_USERNAME}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }
        stage('Generate Dynamic Playbook') {
            steps {
                script {
                    def playbookTemplate = readFile(env.ANSIBLE_PLAYBOOK_PATH)

                    def dynamicPlaybook = playbookTemplate
                        .replace('{{docker_username}}', env.DOCKER_USERNAME)
                        .replace('{{docker_image_name}}', env.DOCKER_IMAGE_NAME)
                        .replace('{{docker_image_tag}}', env.DOCKER_IMAGE_TAG)

                    writeFile file: env.DYNAMIC_PLAYBOOK_PATH, text: dynamicPlaybook
                }
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                withCredentials([string(credentialsId: 'jenkins_sudo_password', variable: 'SUDO_PASSWORD')]) {
                    script {
                        // Corrected line with Groovy interpolation
                       def extraVars = "--extra-vars \"ansible_become_password=${SUDO_PASSWORD}\""
                        sh " sudo ansible-playbook -b ${extraVars} ${env.DYNAMIC_PLAYBOOK_PATH}"
                    }
                }
            }
        }
        stage('Preprocess Kubernetes YAML') {
            steps {
                script {
                    def rawDeploymentYaml = readFile("${env.K8S_DEPLOYMENT_FILE}")  
                    def processedDeploymentYaml = rawDeploymentYaml
                        .replace('${DOCKER_USERNAME}', env.DOCKER_USERNAME)  
                        .replace('${DOCKER_IMAGE_NAME}', env.DOCKER_IMAGE_NAME)
                        .replace('${DOCKER_IMAGE_TAG}', env.DOCKER_IMAGE_TAG)
                    
                    writeFile file: 'processed_deployment.yaml', text: processedDeploymentYaml  
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig_id', variable: 'KUBECONFIG')]) {
                    sh 'kubectl get nodes'  
                    sh 'kubectl apply -f processed_deployment.yaml'  
                }
            }
        }


        
        
        
    }
}

