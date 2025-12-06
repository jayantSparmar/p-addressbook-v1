pipeline {
    agent none

    tools{
        maven 'mymaven'
    }

       parameters {

        string(name: 'ENV', defaultValue: 'Test', description: 'Version to deploy ')

        booleanParam(name: 'excuteTests', defaultValue: true, description: 'Decide to run test cases')

        choice(name: 'APPVERSION', choices: ['1.1', '1.2', '1.3'], description: 'Select application server')

    }

    environment{
             BUILD_SERVER = 'ec2-user@172.31.14.254' 
             DEPLOY_SERVER = 'ec2-user@172.31.11.27' 
             IMAGE_NAME = 'devopsjayant/'
    }  

    stages {
        
        stage('compile') {
            agent any
            steps {
                echo "compile the code ${params.ENV}"
                sh "mvn compile"
            }
        }

        stage('codeReview') {
            agent any
            steps {
                echo 'Reviewing the code for best practise'
                sh "mvn pmd:pmd"
            }
        }

        stage('test') {
            agent any

            when{
                expression {
                    params.excuteTests == true}
            }
            steps {
                echo 'Unit testing of the application'
                sh "mvn test"
            }
         post{
            always{
                junit 'target/surefire-reports/*.xml'
            }
         }
        
        }

        stage('coverageAnalysis') {
           agent {label 'linux_slave'}
            steps {
                echo 'coverage analysis of the test'
                sh "mvn verify"
            }
        }

        // stage('package') {
        //     agent any
        //     steps {
        //         script {
        //     sshagent (['slave2']) {
        //         echo "Packaging the code ${params.APPVERSION}"          
        //         sh "scp -o StrictHostKeyChecking=no my-server-script.sh ${BUILD_SERVER}:/home/ec2-user/"
        //         sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} 'bash ~/my-server-script.sh'"
        //     }
        // }
        //         echo 'packaging the code'
        //         sh "mvn package"
        //     }
        // }


    stage('Containerise the code n push the image to dockerhub') {
            agent any
            steps {
                script{
                sshagent(['slave2']) {
                echo 'Packaging the code'
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'password', usernameVariable: 'devopsjayant')]) {
                sh "scp -o StrictHostKeyChecking=no my-server-script.sh ${BUILD_SERVER}:/home/ec2-user/"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} bash /home/ec2-user/my-server-script.sh ${IMAGE_NAME}"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} sudo docker login -u ${username} -p ${password}"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} sudo docker push ${IMAGE_NAME}"
            
                    }
                }
            }
        }
    }

        stage('Deploy the docker image') {
            agent any
            steps {
                script{
                sshagent(['slave2']) {
                echo 'Packaging the code'
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'password', usernameVariable: 'username')]) {
                //sh "scp -o StrictHostKeyChecking=no server-script.sh ${BUILD_SERVER}:/home/ec2-user/"
                //sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} bash /home/ec2-user/server-script.sh ${IMAGE_NAME}"
                sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} sudo yum install docker -y"
                sh "ssh  ${DEPLOY_SERVER} sudo service docker start"
                sh "ssh  ${DEPLOY_SERVER} sudo docker login -u ${username} -p ${password}"
                sh "ssh  ${DEPLOY_SERVER} sudo docker run -itd -P ${IMAGE_NAME}"
                    }
                }
            }
        }

        // stage('publish') {
        //     agent any
        //     input{
        //         message "Do you want to publish the artifacts to jfrog?"
        //         ok "Yes, publish"
        //     }
        //     steps {
        //         echo "publishing artifacts to jfrog ${params.APPVERSION}"
        //         sh "mvn -U deploy -s settings.xml"
        //     }
        // }
    }
}

