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
        string(name: 'Message', defaultValue: 'this is message', description: 'Message')
        string(name: 'Person', defaultValue: 'dinhlehoang', description: 'Person')
        booleanParam(name: 'willBuild', defaultValue: true, description: 'Build or not?')

        password(name: 'Password', defaultValue: '', description: 'Enter a password')

        string(name: 'MYNAME', defaultValue: 'BI', description: 'my name is BI')
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
                    demo.echoParameters("${params.Username}", "${params.Person}", "${params.willBuild}", "${params.Password}", "${params.MYNAME}" )
                }
                // echo "this is message from my library: ${message}"
                // sh 'mvn --version'
                // echo "Hello ${person}"
                // echo "willBuild: ${willBuild}"
                // echo "Password :${password}"
                // echo "MyName: ${MYNAME}"
                // library 'my-library'
                // script { 
                //     demo ('hello hoang', 'abc', true)
                // }

            }
        }
        stage('Build in maven') {
            // when {
            //     environment name:'isBuilt', value:'true'
            // }
            failFast false
            parallel {
                stage('In parallel 1') {
                    when {
                        expression { return params.willBuild }
                    }
                    // options {
                    //     timeout(time: 20, unit: 'SECONDS')
                    // }
                    steps {
                        // script {
                        //     build()
                        // }
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
                        // script {
                        //     build()
                        // }
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

