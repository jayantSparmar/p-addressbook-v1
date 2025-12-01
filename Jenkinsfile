pipeline {
    agent none

    tools{
        maven 'mymaven'
    }

       parameters {

        string(name: 'ENV', defaultValue: 'TEst', description: 'Version to deploy ')

        booleanParam(name: 'excuteTests', defaultValue: true, description: 'Decide to run test cases')

        choice(name: 'APPVERSION', choices: ['1.1', '1.2', '1.3'], description: 'Select application server')

    }

    environment{
             BUILD_SERVER = 'ec2-user@172.31.7.98'
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

        stage('package') {
            agent any
            steps {
                script {
            sshagent (['slave2']) {
                echo "Packaging the code ${params.APPVERSION}"          
                sh "scp -o StrictHostKeyChecking=no my-server-script.sh ec2-user@${BUILD_SERVER}:/home/ec2-user/"
                sh "sh -o StrictHostKeyChecking=no ec2-user@${BUILD_SERVER} bash ~/my-server-script.sh"
            }
        }
                echo 'packaging the code'
                sh "mvn package"
            }
        }

        stage('publish') {
            agent any
            input{
                message "Do you want to publish the artifacts to jfrog?"
                ok "Yes, publish"
            }
            steps {
                echo "publishing artifacts to jfrog ${params.APPVERSION}"
                sh "mvn -U deploy -s settings.xml"
            }
        }
    }
}

