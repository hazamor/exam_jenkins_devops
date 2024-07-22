pipeline {
environment {
NODE_PORT_DEV = 30001 // deploy manually in dev when it is feature brach
NODE_PORT_QA = 30002 // deploy automatically in qa when it is the main brach
NODE_PORT_STAGING = 30003 // deploy automatically in staging when it is release brach (name like release)
NODE_PORT_PROD = 30004 // deploy manually in prod when it is release brach (name like release)
DOCKER_ID = "hazamor" 
DOCKER_IMAGE_MOVIES = "jenkins-movies"
DOCKER_IMAGE_CAST = "jenkins-cast"
DOCKER_TAG = "${env.GIT_BRANCH ==~ /.*release.*/ ? 'lastest' : env.GIT_COMMIT}"
} 

agent any 
stages {
        stage(' Docker Build'){
            steps {
                script {
                sh '''
                 echo $GIT_BRANCH
                 echo $TAG_NAME
                 echo "DOCKER_TAG = ${DOCKER_TAG}"
                 docker rm -f moviescontainer
                 docker rm -f castcontainer
                 docker build -t "$DOCKER_ID/$DOCKER_IMAGE_MOVIES:$DOCKER_TAG" ./movie-service
                 docker build -t "$DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG" ./cast-service
                '''
                }
            }
        }
        stage('Docker run'){ 
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

        stage('Deploy dev'){ // deploy manually in dev when it is feature brach
            environment
            {
                KUBECONFIG = credentials("config") 
            }

            when {
                expression { GIT_BRANCH ==~ /(feature)/ }
            }
            steps {
                 timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in dev ?', ok: 'Yes'
                }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install app ./app --values=./app/values.yaml --namespace dev\
                --set image.movies.repository="$DOCKER_ID/$DOCKER_IMAGE_MOVIES" \
                --set image.movies.tag="$DOCKER_TAG"  \
                --set image.cast.repository="$DOCKER_ID/$DOCKER_IMAGE_CAST" \
                --set image.cast.tag="$DOCKER_TAG" \
                --set service.nginxservice.nodeport="$NODE_PORT_DEV" 
                '''
                }
            } 

        }

        stage('Deploy QA'){ // deploy automatically in qa when it is the main brach
            environment
            {
                KUBECONFIG = credentials("config") 
            }

            when {
                expression { GIT_BRANCH ==~ /.*main.*/ }
            }

            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install app ./app --values=./app/values.yaml --namespace qa\
                --set image.movies.repository="$DOCKER_ID/$DOCKER_IMAGE_MOVIES" \
                --set image.movies.tag="$DOCKER_TAG"  \
                --set image.cast.repository="$DOCKER_ID/$DOCKER_IMAGE_CAST" \
                --set image.cast.tag="$DOCKER_TAG" \
                --set service.nginxservice.nodeport="$NODE_PORT_QA" 
                '''
                }
            } 

        }

        stage('Deploy staging'){  // deploy automatically in staging when it is release brach (name like release)
            environment
            {
                KUBECONFIG = credentials("config") 
            }

            when {
                expression { GIT_BRANCH ==~ /.*release.*/ }
            }

            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install app ./app --values=./app/values.yaml --namespace staging\
                --set image.movies.repository="$DOCKER_ID/$DOCKER_IMAGE_MOVIES" \
                --set image.movies.tag="$DOCKER_TAG"  \
                --set image.cast.repository="$DOCKER_ID/$DOCKER_IMAGE_CAST" \
                --set image.cast.tag="$DOCKER_TAG" \
                --set service.nginxservice.nodeport="$NODE_PORT_STAGING" 
                '''
                }
            } 

        }


        stage('Deploy prod'){ // deploy manually in prod when it is release brach (name like release)
            environment
            {
                KUBECONFIG = credentials("config") 
            }

            when {
                expression { GIT_BRANCH ==~ /.*release.*/ }
            }

            steps {
                 timeout(time: 60, unit: "MINUTES") {
                        input message: 'Do you want to deploy in prod ?', ok: 'Yes'
                }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                helm upgrade --install app ./app --values=./app/values.yaml --namespace prod\
                --set image.movies.repository="$DOCKER_ID/$DOCKER_IMAGE_MOVIES" \
                --set image.movies.tag="$DOCKER_TAG"  \
                --set image.cast.repository="$DOCKER_ID/$DOCKER_IMAGE_CAST" \
                --set image.cast.tag="$DOCKER_TAG" \
                --set service.nginxservice.nodeport="$NODE_PORT_PROD" 
                '''
                }
            } 

        }

}
}