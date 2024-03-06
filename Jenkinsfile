pipeline{
    agent none
    // tools{ maven 'maven 3.6.1' } 
    stages {
        stage('worker-build') {
            when { changeset "**/worker/**" }
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Compiling worker app'
                dir('worker'){
                    sh 'mvn compile'
                }
            }
        }
        stage('worker-test') {
            when { changeset "**/worker/**" }
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Running unit tests on worker app'
                dir('worker'){
                    sh 'mvn clean test'
                }
            }
        }
        stage('worker-package') {   
            when { 
                changeset "**/worker/**"
                branch 'master'
            }
            agent{
                docker{
                    image 'maven:3.6.1-jdk-8-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo 'Packaging worker app'
                dir('worker'){
                    sh 'mvn package -DskipTests'
                }
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
        stage('worker-docker-package') {
            when { 
                changeset "**/worker/**"
                branch 'master'
            }
            agent any
            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/','dockerlogin') {
                        def workerImage = docker.build ("${env.GIT_URL.tokenize('/.')[-3]}/worker:v${env.BUILD_ID}", './worker')
                        workerImage.push()
                        workerImage.push('latest')
                    }
                }
            }
        }
        stage('vote-test'){
            when {
                changeset '**/vote/**'
            }
            agent {
                docker {
                    image 'python:2.7-alpine'
                    args '-u root --privileged'
                }
            }
            steps{
                echo 'Building vote app'
                dir('vote'){
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests tests/'
                }
            }
        }
        stage('vote-integration-tests') {
            when {
                changeset '**/vote/**'
            }
            agent any
            steps{
                echo 'Running integration tests on vote'
                dir('vote'){
                    sh 'sh integration_test.sh'
                }
            }
        }
        stage('vote-docker-package') {
            when {
                changeset '**/vote/**'
                branch 'master'
            }
            agent any
            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/','dockerlogin') {
                        def workerImage = docker.build ("${env.GIT_URL.tokenize('/.')[-3]}/vote:v${env.BUILD_ID}", './vote')
                        workerImage.push()
                        workerImage.push('latest')
                    }
                }
            }
        }
        stage('result-build') {
            when { changeset "**/result/**" }
            agent{
                docker{
                    image 'node:8.9.0-alpine'
                }
            }
            steps {
                echo 'Building node app'
                dir('result'){
                    sh 'npm install'
                    sh 'npm ls'
                }
            }
        }
        stage('result-test') {
            when { changeset "**/result/**" }
            agent{
                docker{
                    image 'node:8.9.0-alpine'
                }
            }
            steps {
                echo 'Running unit tests on result app'
                dir('result'){
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('result-docker-package') {
            when { 
                changeset "**/worker/**"
                branch 'master'
            }
            agent any
            steps {
                echo 'Packaging result app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/','dockerlogin') {
                        def workerImage = docker.build ("${env.GIT_URL.tokenize('/.')[-3]}/result:v${env.BUILD_ID}", './result')
                        workerImage.push()
                        workerImage.push('latest')
                    }
                }
            }
        }
        stage('Sonarqube') {
            agent any
            // when {
            //     branch 'master'
            // }
            environment {
                sonarpath = tool 'SonarScanner' //sonar installation
            }
            steps {
                echo 'Running Sonarqube Analysis'
                withSonarQubeEnv('sonar-pipilacha'){ //global sonar server config
                    sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
                }
            }
        }
        stage("Quality Gate") {
            agent any
            steps {
                echo 'Waiting for Quality Gate!'
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('deploy-instavote-to-dev') {
            agent any
            when {
                branch 'master'
            }
            steps {
                echo 'Deploying instavote with docker compose'
                sh 'docker-compose up -d'
            }
        }
        stage('Triggering ArgoCD deployment') {
            agent any
            // when {
            //     branch 'master'
            // }
            steps {
                echo "Git commit ${env.GIT_COMMIT}"
                echo "Trigerring argocd"
                // passing variables to job deployment run by instavote-deploy repository Jenkinsfile
                build job: 'deployment', parameters: [string(name: 'DOCKERTAG', value: GIT_COMMIT)]
            }
        }
    }
    post{
        always{            
            echo 'Worker multibranch pipeline completed!'
        }
        failure{
            slackSend (message: "Build failed: ${env.JOB_NAME} ${BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success{
            slackSend (message: "Build sucess: ${env.JOB_NAME} ${BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
    }
}
