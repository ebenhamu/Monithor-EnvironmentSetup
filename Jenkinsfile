@NonCPS
List<String> getChangedFilesList() {
    def changedFiles = []
    for (changeLogSet in currentBuild.changeSets) {
        for (entry in changeLogSet.getItems()) { // for each commit in the detected changes
            for (file in entry.getAffectedFiles()) {
                changedFiles.add(file.getPath()) // add changed file to list
            }
        }
    }
    return changedFiles
}

pipeline {
    agent any

    environment {
        TF_REPO_URL = "https://github.com/ebenhamu/Monithor-infrastructure.git"
        TF_REPO_DIR = "~/jobs/MoniThorDeployment/workspace/Monithor-infrastructure"
        TF_BRANCH = "1.0.0"  // Specify the branch
        GITHUB_CREDENTIALS = credentials('monithor_git_token') // Replace with your credential ID
    }

    stages {
        stage('Check Changed Files') {
            steps {
                script {
                    def changedFiles = getChangedFilesList()
                    def filesOutput = changedFiles.join('\n')
                    echo "Changed Files:\n${filesOutput}"
                    echo "file only ${file.getPath().tokenize('/').last()}"
                }
            }
        }

        stage('Extract Node Counts') {
            steps {
                script {
                    def changedFiles = getChangedFilesList()
                    def controlPlaneCount = 0
                    def workerNodeCount = 0

                    for (file in changedFiles) {
                        if (file.getPath().tokenize('/').last() == 'manifest.json') {
                            echo "Processing JSON file: ${file}"                            
                            // try {
                            //     def jsonContent = readJSON file: file
                            //     if (jsonContent.control_plane?.count) {
                            //         controlPlaneCount = jsonContent.control_plane.count
                            //     }
                            //     if (jsonContent.worker_nodes?.count) {
                            //         workerNodeCount = jsonContent.worker_nodes.count
                            //     }
                            //     } catch (Exception e) {
                            //         echo "Failed to process JSON file: ${file}"
                            //         echo "Error: ${e.message}"
                            //     }
                        // def jsonContent = readJSON file : "${file}"
                            // if (jsonContent.control_plane) {
                            //     controlPlaneCount = jsonContent.control_plane.count
                            // }
                            // if (jsonContent.worker_nodes) {
                            //     workerNodeCount = jsonContent.worker_nodes.count
                            // }
                        }
                    }

                    echo "Control Plane Count: ${controlPlaneCount}"
                    echo "Worker Node Count: ${workerNodeCount}"

                    // Save the counts as environment variables for later stages
                    env.CONTROL_PLANE_COUNT = controlPlaneCount.toString()
                    env.WORKER_NODE_COUNT = workerNodeCount.toString()
                }
            }
        }

 




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


        stage('Update Terraform Config') {
            steps {
                script {
                    echo "Updating Terraform node counts..."
                    sh """
                        sed -i 's/count         = [0-9]*/count         = ${env.CONTROL_PLANE_COUNT}/' ${TF_REPO_DIR}/terraform/main.tf
                        sed -i 's/count         = [0-9]*/count         = ${env.WORKER_NODE_COUNT}/' ${TF_REPO_DIR}/terraform/main.tf
                    """
                }
            }
        }

        stage('Update Terraform Config 1') {
            steps {
                script {
                    echo "Updating Terraform configuration with node counts..."
                    sh """
                        cd ${TF_REPO_DIR}
                        echo 'control_plane_count = ${env.CONTROL_PLANE_COUNT}' > terraform.tfvars
                        echo 'worker_node_count = ${env.WORKER_NODE_COUNT}' >> terraform.tfvars
                    """
                }
            }
        }
    }
}
