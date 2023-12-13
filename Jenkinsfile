node {
    checkout scm
    withEnv(['HOME=.']) {
        docker.image('docker:18.09-dind').withRun(""" --privileged """) { dindContainer ->
            docker.withRegistry('', 'credentials-id') {    
                stage ('Build and Upload') {
                    script {
                        def myNetwork = 'my-network'
                        
                        sh """
                            docker network create $myNetwork
                            cd src
                            docker-compose --host tcp://docker:2375 build
                            docker-compose --host tcp://docker:2375 up -d
                            docker --host tcp://docker:2375 images
                            cd ..
                            
                            # Continue with your upload steps
                            cp -RT app /app/src/workspace
                            cd /app/src/workspace
                            ie-app-publisher-linux de c -u http://docker:2375
                            export IE_SKIP_CERTIFICATE=true
                            ie-app-publisher-linux em li -u "$IEM_URL" -e $USER_NAME -p $PSWD
                            ie-app-publisher-linux em app cuv -a $APP_ID -v 0.0.$BUILD_NUMBER -y ./docker-compose.prod.yml -n '{"hello-edge":[{"name":"hello-edge","protocol":"HTTP","port":"80","headers":"","rewriteTarget":"/"}]}' -s 'hello-edge' -t 'FromBoxReverseProxy' -u "hello-edge" -r "/"
                            ie-app-publisher-linux em app uuv -a $APP_ID -v 0.0.$BUILD_NUMBER
                        """
                    }
                }        
            }
        }
    }
}
