@Library('my-library') _

pipeline {
    agent any

    tools { 
        maven 'my-maven' 
    }
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '3', numToKeepStr: '3')
    }

    parameters {
        string(name: 'message', defaultValue: 'this is message', description: 'Message')
        string(name: 'person', defaultValue: 'dinhlehoang', description: 'Person')
        booleanParam(name: 'willBuild', defaultValue: true, description: 'Build or not?')

        password(name: 'password', defaultValue: '', description: 'Enter a password')

        string(name: 'myname', defaultValue: 'BI', description: 'my name is BI')
        booleanParam(name: 'readsecret', defaultValue: 'true')
        booleanParam(name: 'readtext', defaultValue: 'true')
    }
    environment {
        DOCKERHUB_CREDENTIALS=credentials('dockerhub')
        HOVATEN = 'DINHLEHOANG'
        abc = 'asdf'
        name = 'dinhlehoang'
    }
    stages {

        stage('Initialize') {
            when {
                allOf {
                    branch 'develop'
                    environment name:'name', value:'dinhlehoang'
                }
            }
            agent {
                docker {
                    image 'maven:latest'
                }
            }
            steps {
                script {
                    demo.call()
                    // demo.echoParameters("abcd", "mnqp", true, "thisispassword", "dinhlehoang")
                }

            }
        }
        stage('Build in maven') {
            failFast false
            parallel {
                stage('In parallel 1') {
                    steps {
                        script {
                            demo.build()
                        }
                        
                    }
                }
                stage('In parallel 2') {
                    agent any 
                    steps {
                        script {
                            demo.build()
                        }

                    }
                }
            }
        }

        stage('Package to docker image') {

            steps {
                unstash 'app' 
                sh 'docker build -t hoangledinh65/springboot-image:1.0 .'
            }
        }
        stage('Pushing image') {
            steps {
                echo 'Start pushing.. with credential'
                sh 'echo $DOCKERHUB_CREDENTIALS'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push hoangledinh65/springboot-image:1.0'
                
            }
        }
        stage('Deploying and Cleaning') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image rm hoangledinh65/springboot-image:1.0 || echo "this image does not exist" '
                sh 'docker container stop my-demo-springboot || echo "this container does not exist" '
                sh 'docker network create jenkins || echo "this network exists"'
                sh 'echo y | docker container prune '
                sh 'echo y | docker image prune'
                sh 'whoami'
                sh 'docker container run -d --rm --name my-demo-springboot -p 8082:8080 --network jenkins hoangledinh65/springboot-image:1.0'
            }
        }
        stage('Read file') {
            when {
                anyOf {
                    environment name:'readsecret', value: 'true'
                    environment name:'readtext', value: 'true'
                }
            }
            stages {
                stage ('Readsecret') {
                    agent any 
                    steps {
                        echo 'abcd'
                        // withCredentials([
                        //     file(credentialsId: 'demosecret', variable: 'demosecret')
				        // ]) { 
				    	//         sh "cat $demosecret"
                        // }
                    }
                }

            }
    }

}
    post {
        always {
            echo 'post office'
        }
    }
}

