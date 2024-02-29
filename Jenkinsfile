pipeline {
    agent any

    stages {
        stage('one') {
            steps {
                echo 'step 1'
                sleep 5
            }
        }
        stage('two') {
            steps {
                echo 'step 2'
                sleep 3
            }
        }
        stage('three') {
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
