pipeline {
    agent any

    tools { 
        maven 'my-maven' 
        jdk 'my-jdk' 
    }
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '3', numToKeepStr: '3')
    }

    parameters {
        string(name: 'Username', defaultValue: 'dinhlehoang', description: 'Username')

        booleanParam(name: 'willBuild', defaultValue: true, description: 'Build or not?')

        password(name: 'Password', defaultValue: '', description: 'Enter a password')

        string(name: 'MYNAME', defaultValue: 'BI', description: 'my name is BI')
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
                echo "${env.name}"
                // sh 'sudo apt install maven'
                echo "Hello ${params.PERSON}"

                echo "Biography: ${params.BIOGRAPHY}"

                echo "Toggle: ${params.TOGGLE}"

                echo "Choice: ${params.CHOICE}"

                echo "Password: ${params.PASSWORD}"
                echo "Myname is : ${params.MYNAME}"
                // sh 'whoami'
                sh 'mvn --version'

            }
        }
        stage('Build in maven') {
            // when {
            //     environment name:'isBuilt', value:'true'
            // }
            parallel {
                stage('In parallel 1') {
                    agent {
                        label 'aws-amz'
                    }
                    steps {
                        sh ''' 
                            echo "in parallel 1"
                            mvn clean package
                            '''
                        stash includes: 'target/*', name: 'app'
                    }
                }
                stage('In parallel 2') {
                    agent any 
                    steps {
                        sh ''' 
                            echo "in parallel 2"
                            mvn clean package
                            '''
                        stash includes: 'target/*', name: 'app'

                    }
                }
            }
        }

        stage('Package to docker image') {

            steps {
                unstash 'app' 
                sh 'ls -la'
                sh 'ls -la target'
                sh 'docker build -t hoangledinh65/springboot-image:1.0 .'
            }
        }
        stage('Pushing image') {
            steps {
                sh 'echo $HOVATEN'
                echo 'Start pushing.. with credential'
                sh 'echo $DOCKERHUB_CREDENTIALS'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push hoangledinh65/springboot-image:1.0'
                
            }
        }
        stage('Deploying and Cleaning') {
            // agent {
            //     node {
            //         label 'ubuntu'
            //     }
            // }
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

    }
}
