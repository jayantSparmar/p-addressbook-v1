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
            agent any
            steps {
                echo 'coverage analysis of the test'
                sh "mvn verify"
            }
        }

        stage('package') {
            agent {label 'linux_slave'}
            steps {
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

