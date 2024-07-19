pipeline {
environment {
DOCKER_ID = "hazamor" 
DOCKER_IMAGE_MOVIES = "jenkins-movies"
DOCKER_IMAGE_CAST = "jenkins-cast"
DOCKER_TAG = "v.${BUILD_ID}.0" // 
D_TAG = "v.${BUILD_ID}.0"
}

agent any // Jenkins will be able to select all available agents
stages {
        stage(' Set env vars'){ 
            when {
                branch 'master'
            }
            environment {
                D_TAG = $GIT_COMMIT
            }
            steps {
                 sh 'print $D_TAG'
            }
        }
        stage(' Docker Build'){ // docker build image stage 
            steps {
                script {
                sh '''
                 echo $D_TAG
                 docker rm -f moviescontainer
                 docker rm -f castcontainer
                 docker build -t "$DOCKER_ID/$DOCKER_IMAGE_MOVIES:$DOCKER_TAG" ./movie-service
                 docker build -t "$DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG" ./cast-service
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 8001:8000 --name moviescontainer "$DOCKER_ID/$DOCKER_IMAGE_MOVIES:$DOCKER_TAG"
                    docker run -d -p 8002:8000 --name castcontainer "$DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG"
                    sleep 10
                    '''
                    }
                }
            }

        stage('Docker Push'){ 
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push "$DOCKER_ID/$DOCKER_IMAGE_MOVIES:$DOCKER_TAG"
                docker push "$DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG"
                '''
                }
            }

        }

    
}
}