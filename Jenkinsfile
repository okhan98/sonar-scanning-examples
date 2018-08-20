/*
Author - jruben@roostify.com
Date   - 04/20/2018

This is a common jenkins pipeline for Java microservices.

The sequence of stages are
    1. Checkout     -- Checksout code from GIT.
    2. Clean        -- Cleans project directory.
    3. compile      -- Compile code.
    4. Build        -- Build code minus the tests
    5. Test         -- Run unit tests
    6. Dockerize    -- Create a docker image of the project
    7. Docker push  -- Push the created container to Amaxon ECR.

*/
pipeline {
    agent any

    environment {
            AWS_ECR_LOGIN='true'
    }
    stages {
        stage ('checkout') {
            steps {
                echo 'Checking out code from git repo'
                checkout scm
            }
        }

        stage ('clean') {
            steps {
                sh "./gradlew clean"
            }
        }

        stage ('compile') {
            steps {
                sh "./gradlew compileJava"
            }
        }

        stage ('Build') {
            steps {
                sh "./gradlew build -x test -i"
            }
        }

        stage ('test') {
            steps {
                sh "./gradlew test"
            }
        }

        stage('Dockerize') {
            steps {
                // The below eval was added to escape the 12 hour login issue window on AWS. This will not get new credentials on every build on Jenkins.
                sh "eval `aws ecr get-login --no-include-email --region us-west-2 | sed 's|https://||'`"
                sh "./gradlew createDockerImage"
            }
        }

        /*
        The ECR credentials are present in the global variables of Jenkins.
        This way this code can work even when the credentials change.
        */
        stage('Docker Push') {
            steps {
                script {
                    def registryUrl = "${CUSTOM_DOCKER_REGISTRY_URL}"
                    def registryCredentialsId = "${CUSTOM_DOCKER_REGISTRY_CREDENTIAL}"
                    def buildTag = computeBuildTag()
                    docker.withRegistry(registryUrl,registryCredentialsId) {
                        docker.image(getDockerImageName()).push(buildTag)
                    }
                }
            }
        }
    }
}

/*
This method will create a tag for the docker image from the GIT branch and Jenkins build number
Example format is origin-feature-jj-jenkins-23
*/
def computeBuildTag(){
    def buildTag = "${env.GIT_BRANCH}-${BUILD_NUMBER}"
    buildTag = buildTag.replaceAll('\\/','-')
    return buildTag
}

/*
This Method is a place holder for the image name.
Every new microservice project will need to overide this name to suite the app name.
*/
def getDockerImageName(){
    return 'roostify/report-service'
}