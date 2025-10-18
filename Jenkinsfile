pipeline {
    agent any

    tools {
        maven 'M2_HOME'   // Maven tool configured in Jenkins
    }

    environment {
        TAG_NAME = "3.0"
    }

    stages {

        stage('Git Checkout') {
            steps {
                echo 'Cloning the repo from GitHub'
                git branch: 'master', url: 'https://github.com/Chandrika-git05/star-agile-insurance-project.git'
            }
        }

        stage('Build & Package') {
            steps {
                echo 'Compiling, testing, and packaging the application'
                sh 'mvn clean package'
            }
        }

        stage('Publish Test Reports') {
            steps {
                echo 'Publishing test report'
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: 'target/surefire-reports',
                    reportFiles: 'index.html',
                    reportName: 'HTML Report'
                ])
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Creating Docker image'
                sh "sudo docker build -t chandrika5592/insureme:${TAG_NAME} ."
            }
        }

        stage('Login & Push to DockerHub') {
            steps {
                echo 'Logging into DockerHub and pushing image'
               withCredentials([usernamePassword(credentialsId: 'dockercreds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
    sh """
        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
        docker push chandrika5592/insureme:3.0
    """
}

                }
            }
        }

        stage('Deploy to Test Server via Ansible') {
            steps {
                echo 'Deploying the application using Ansible'
                ansiblePlaybook(
                    become: true,
                    credentialsId: 'ansible_ssh',     // SSH credential ID from Jenkins
                    disableHostKeyChecking: true,
                    installation: 'ansible',          // Jenkins Ansible installation name
                    inventory: '/etc/ansible/hosts',  // Path to your inventory file
                    playbook: 'ansible-playbook.yml'  // Your Ansible playbook
                )
            }
        }
    }

    post {
        failure {
            emailext(
                to: 'chandrikashrikrishna@gmail.com',
                subject: "Job ${env.JOB_NAME} #${env.BUILD_NUMBER} Failed 🚨",
                body: """<p>Dear Chandrika,</p>
                         <p>The Jenkins job <b>${env.JOB_NAME}</b> (Build #${env.BUILD_NUMBER}) has failed.</p>
                         <p>Please check the logs: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                         <p>Regards,<br>Jenkins CI</p>"""
            )
        }

        success {
            emailext(
                to: 'chandrikashrikrishna@gmail.com',
                subject: "Job ${env.JOB_NAME} #${env.BUILD_NUMBER} Succeeded ✅",
                body: """<p>Dear Chandrika,</p>
                         <p>The Jenkins job <b>${env.JOB_NAME}</b> (Build #${env.BUILD_NUMBER}) completed successfully.</p>
                         <p>You can review the build here: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                         <p>Regards,<br>Jenkins CI</p>"""
            )
        }
    }
}
