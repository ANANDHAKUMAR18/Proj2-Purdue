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
        SERVICE_YAML='kubernetes/service.yaml'
        LABEL='abc-app' 
        GRAFANA_DASHBOARD_URL='http://18.232.180.202:3000/'
        PROMETHEUS_CONFIG_FILE='/etc/systemd/system/prometheus.service'
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
                    sh " echo ${SUDO_PASSWORD} | sudo -S docker build -t ${DOCKER_USERNAME}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
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
                        sh "echo ${SUDO_PASSWORD} | sudo -S docker login -u ${DOCKER_USERNAME} -p ${env.DOCKER_PASSWORD} ${DOCKER_REGISTRY}"
                        sh " echo ${SUDO_PASSWORD} | sudo -S docker push ${DOCKER_USERNAME}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
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
                        sh " echo ${SUDO_PASSWORD} | sudo -S ansible-playbook -b ${extraVars} ${env.DYNAMIC_PLAYBOOK_PATH}"
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
                        .replace('${LABEL}' , env.LABEL)
                    writeFile file: 'processed_deployment.yaml', text: processedDeploymentYaml  
                }
            }
        }
        stage('Preprocess Service YAML') {
            steps {
                script {
                    def rawServiceYaml = readFile("${env.SERVICE_YAML}")  
                    def processedServiceYaml = rawServiceYaml
                        .replace('${DOCKER_USERNAME}', env.DOCKER_USERNAME)  
                        .replace('${DOCKER_IMAGE_NAME}', env.DOCKER_IMAGE_NAME)
                        .replace('${DOCKER_IMAGE_TAG}', env.DOCKER_IMAGE_TAG)
                        .replace('${LABEL}' , env.LABEL)
                    writeFile file: 'processed_service.yaml', text: processedServiceYaml  
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig_id', variable: 'KUBECONFIG')]) {
                    sh 'kubectl get nodes'  
                    sh 'kubectl apply -f processed_deployment.yaml'  
                    sh 'kubectl apply -f processed_service.yaml'
                }
            }
        }
        stage('Install Node Exporter') {
            steps {
                // Add steps to install Node Exporter on the target machine
                // For example, using a shell script
                
                sh 'wget https://github.com/prometheus/node_exporter/releases/download/v0.16.0-rc.1/node_exporter-0.16.0-rc.1.linux-amd64.tar.gz'
                sh 'tar -xvzf node_exporter-0.16.0-rc.1.linux-amd64.tar.gz'
                
            }
        }
       
        stage('Write and Append File') {
            steps { 
                withCredentials([string(credentialsId: 'jenkins_sudo_password', variable: 'SUDO_PASSWORD')]) {
                  script {
                      // Define the content of the file
                      def fileContent = """
                      [Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
User=root
Group=root
ExecStart=/var/lib/jenkins/workspace/Edureka/node_exporter-0.16.0-rc.1.linux-amd64/node_exporter
[Install]
WantedBy=default.target

                    """

                      writeFile file: 'node_exporter.service', text: fileContent

                    
                      sh 'echo ${SUDO_PASSWORD} | sudo -S scp node_exporter.service /etc/systemd/system'

                      // Clean up the file after appending
                      sh 'rm node_exporter.service'
                      sh 'rm -rf  node_exporter-0.16.0-rc.1.linux-amd64.tar.gz'
                  }
               }   
            }
        }
        stage('Starting Node Exporter') {
            steps{
                withCredentials([string(credentialsId: 'jenkins_sudo_password', variable: 'SUDO_PASSWORD')]) {
                   script {
                      sh 'echo ${SUDO_PASSWORD} | sudo -S systemctl enable node_exporter'
                      sh 'echo ${SUDO_PASSWORD} | sudo -S systemctl start node_exporter'
                      sh 'echo ${SUDO_PASSWORD} | sudo -S systemctl status node_exporter'
                   }
                }
            }
        }
        stage('Monitor with Grafana') {
            steps {
                echo "Access Grafana Dashboard at ${GRAFANA_DASHBOARD_URL}"
            }
        }


        
        
        
    }
}

