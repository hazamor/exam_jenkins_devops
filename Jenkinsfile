pipeline {
environment {
DOCKER_ID = "hazamor" 
DOCKER_IMAGE_MOVIES = "jenkins-movies"
DOCKER_IMAGE_CAST = "jenkins-cast"
DOCKER_TAG = "v.${BUILD_ID}.0" // 
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker build -t "$DOCKER_ID/$DOCKER_IMAGE_MOVIES:latest" ./movies
                 docker tag "$DOCKER_ID/$DOCKER_IMAGE_MOVIES:latest" "$DOCKER_ID/$DOCKER_IMAGE_MOVIES:DOCKER_TAG"
                 docker build -t "$DOCKER_ID/$DOCKER_IMAGE_CAST:latest" ./cast
                 docker tag "$DOCKER_ID/$DOCKER_IMAGE_CAST:latest" "$DOCKER_ID/$DOCKER_IMAGE_CAST:DOCKER_TAG"
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 8001:8000 --name castcontainer "$DOCKER_ID/$DOCKER_IMAGE_MOVIES:latest"
                    docker run -d -p 8002:8000 --name castcontainer "$DOCKER_ID/$DOCKER_IMAGE_CAST:latest"
                    sleep 10
                    curl localhost:8001
                    curl localhost:8002
                    docker rm -f moviescontainer
                    docker rm -f castcontainer
                    '''
                    }
                }
            }

        stage('Docker Push'){ //we pass the built image to our docker hub account 
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
                }
            }

        }

    
}
}