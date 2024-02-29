pipeline {
    agent any

    tools{
        maven 'maven 3.6.1'
    }

    stages {
        stage('build') {
            steps {
                echo 'Compiling worker app'
                dir('worker'){
                    sh 'mvn compile'
                }
            }
        }
        stage('test') {
            steps {
                echo 'Running Test suites'
                dir('worker'){
                    sh 'mvn clean test'
                }
            }
        }
        stage('package') {
            when{
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'step 3'
                sleep 9
            }
        }
    }
    post{
        always{
            echo 'This pipeline is completed'
        }
        failure{
            slackSend (message: "Build failed: ${env.JOB_NAME} ${BUILD_NUMBER}")
        }
        success{
            slackSend (message: "Build sucess: ${env.JOB_NAME} ${BUILD_NUMBER}")
        }
    }
}
