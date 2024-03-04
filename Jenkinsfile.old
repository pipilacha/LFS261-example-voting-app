pipeline {
    agent any

    stages {
        stage('one') {
            steps {
                echo 'step 1 stage 1'
            }
        }
        stage('two') {
            steps {
                echo 'step 1 stage 2'
            }
        }
        stage('package') {
            when{
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                eecho 'step 1 stage 3'
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
