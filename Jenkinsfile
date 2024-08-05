pipeline {
    agent any // means any agent
    environment {
        DOCKER_ID = "ngorgol"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    stages {
        stage ('Building'){
            steps{
                sh 'pip install -r requirements.txt'
            }
        }
        stage ('Testing'){
            steps{
                sh 'python -m unittest'
            }
        }
        stage ('Deploying'){
            steps{
                script{
                    sh'''
                    docker rm -f jenkins
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG . 
                    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage ('User acceptance'){
            steps{
                input(message: "Proceed to push to main", ok: "Yes")
            }
        }
        stage ('Pushing and Merging'){
            parallel{
                stage ('Pushing'){
                    environment{
                        DOCKERHUB_CREDENTIALS = credentials("DOCKER_HUB_PASS")
                    }
                    steps{
                        script{
                            sh''' 
                            echo $DOCKERHUB_CREDENTIALS | docker login -u $DOCKER_ID --password-stdin
                            docker image push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG 
                            '''
                        }
                    }
                }
                stage ('Merging'){
                    steps{
                        sh 'echo "Merging done"'
                    }

                }
            }
        }
    }
    post {
        always {
            echo "Disconnect from Dockerhub"
            sh 'docker logout'
        }
    }
}