pipeline {
    agent any

    environment {
        TF_REPO_URL = "https://github.com/ebenhamu/Monithor-infrastructure.git"
        TF_REPO_DIR = "${WORKSPACE}/Monithor-infrastructure"  // Workspace for the repository
        TF_BRANCH = "1.0.0"  // Specify the branch
        GITHUB_CREDENTIALS = credentials('monithor_git_token') // Replace with your credential ID
        TF_VERSION = "1.5.2" // Specify the Terraform version to install
    }

    stages {
        stage('Clone Terraform Repo') {
            steps {
                script {
                    echo "Cloning Terraform repository (branch ${env.TF_BRANCH})..."
                    sh """
                        rm -rf ${TF_REPO_DIR}
                        git clone --branch ${TF_BRANCH} https://${env.GITHUB_CREDENTIALS_USR}:${env.GITHUB_CREDENTIALS_PSW}@github.com/ebenhamu/Monithor-infrastructure.git ${TF_REPO_DIR}
                    """
                }
            }
        }

        stage('Copy Manifest File to Terraform Folder') {
            steps {
                script {
                    def changedFiles = getChangedFilesList()
                    def destinationDir = "${TF_REPO_DIR}/tf"

                    // Ensure target folder exists
                    sh "mkdir -p ${destinationDir}"

                    // Process changed files and copy manifest.json
                    for (file in changedFiles) {
                        if (file.endsWith('manifest.json')) {
                            echo "Copying JSON file: ${file}"
                            sh """
                                cp ${WORKSPACE}/${file} ${destinationDir}
                            """
                        }
                    }
                }
            }
        }

        stage('Install Terraform') {
            steps {
                script {
                    echo "Installing Terraform version ${TF_VERSION}..."
                sh """
                    curl -o terraform.zip https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_amd64.zip
                    unzip -o terraform.zip
                    mkdir -p ${WORKSPACE}/bin
                    mv terraform ${WORKSPACE}/bin/
                    export PATH=${WORKSPACE}/bin:\$PATH
                    terraform -version
                """

                }
            }
        }

          
    
    
        stage('Terraform Init Plan Apply ') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws_credentials']]) {
                    
                    sh """

                    echo "Running Terraform commands in ${TF_REPO_DIR}/tf..."
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    cd ${TF_REPO_DIR}/tf
                    export PATH=/var/jenkins_home/jobs/MoniThorDeployment/workspace/bin:/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
                    terraform init -input=false  # Initialize the Terraform directory
                    terraform plan -out=tfplan   # Generate the plan and output it as tfplan
                    terraform apply -input=false -auto-approve tfplan  # Apply the plan
                    
   
                    """
                }
            }
        }
    
}



    }
}

@NonCPS
List<String> getChangedFilesList() {
    def changedFiles = []
    for (changeLogSet in currentBuild.changeSets) {
        for (entry in changeLogSet.getItems()) {
            for (file in entry.getAffectedFiles()) {
                changedFiles.add(file.getPath())
            }
        }
    }
    return changedFiles
}
