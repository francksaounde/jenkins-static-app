
pipeline {
     environment { 
       //Secrets to set up in Jenkins  
     DOCKERHUB_PASSWORD  = credentials('dockerhub') // We define a username/password ‘credentials’ parameter for connecting to the docker hub.
	     //'dockerhub' is the id used in this definition
      HEROKU_API_KEY = credentials('heroku_api_key')   // to connect to heroku, we use a ‘credentials’ parameter of the text secret type
	     
      // apart from the secrets that we define as global parameters for Jenkins
      // the other variables can be set as ‘string’ parameters in the job or directly in this file
	ID_DOCKER = "dockerhub_account_name"  //replace with your dockerhub account name
		
       //DOCKER_IMG = $IMAGE_NAME //"static_web_img"
	DOCKER_IMG = "static_web_img"
	   
       //TAG = $IMAGE_TAG //"v1"
	TAG = "v1"
	   
	//APP_NAME = $APPLI_NAME  // application name, displayed as is on heroku
	APP_NAME = "static_app"
	   
       //EXT_PORT = $EXTERNAL_PORT
	EXT_PORT = "5000"  // for example
	   
	//INT_PORT = $INTERNAL_PORT 
	INT_PORT = "80"	   //default nginx port
	   
	// localhost or ip address of the machine running the Jenkins stack
	//HOST_ADDRESS = $IP_HOST 
	HOST_ADDRESS = "http://192.168.56.10" 
	   
	SENTENCE_TO_TEST = "A fully responsive site template" //phrase à tester sur la page d'accueil
	   
	// definition of deployment environments	   
       STAGING = "${ID_DOCKER}-staging"
       PRODUCTION = "${ID_DOCKER}-production"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t ${ID_DOCKER}/$DOCKER_IMG:$TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    echo "Clean Environment"
                    docker stop $APP_NAME || echo "container does not exist"
		    docker rm $APP_NAME || echo "container does not exist"
                    docker run --name $APP_NAME -d -p $EXT_PORT:$INT_PORT -e PORT=$INT_PORT ${ID_DOCKER}/$DOCKER_IMG:$TAG
                    sleep 5
                 '''
               }
            }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''	    
    		    # Test of status = 200 and an expression on the home page
	   	    curl -o -I -L -s -w "%{http_code}" $HOST_ADDRESS:${EXT_PORT}
                    curl $HOST_ADDRESS:${EXT_PORT} | grep "$SENTENCE_TO_TEST"
                '''
              }
           }
      }
      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                 docker stop $APP_NAME
                 docker rm $APP_NAME
               '''
             }
          }
     }
     
     stage ('Login and Push Image on docker hub') {	     
        agent any  
          steps {
             script {
               sh '''
	       	   echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
	    	   docker push ${ID_DOCKER}/$DOCKER_IMG:$TAG      
               '''
             }
          }	
      }    
     
      stage('Push image in staging and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/main' }
            }
      agent any
      steps {
          script {
            sh '''
	      # the following line assumes that npm is installed, otherwise install it depending on the linux version (apk, apt, etc )
              npm i -g heroku@7.68.0
	      # sudo npm install -g heroku  
              heroku container:login
              heroku create $STAGING || echo "project already exists"
              heroku container:push -a $STAGING web
              heroku container:release -a $STAGING web
            '''
          }
        }
     }



     stage('Push image in production and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/production' }
            }
      agent any 
      steps {
          script {
            sh '''
	    # the following line assumes that npm is installed, otherwise install it depending on the linux version (apk, apt, etc )
              npm i -g heroku@7.68.0	   
              heroku container:login
              heroku create $PRODUCTION || echo "project already exists"
              heroku container:push -a $PRODUCTION web
              heroku container:release -a $PRODUCTION web
            '''
          }
        }
     }
	 	 
    stage('Clean Docker Image') {
    agent any
	  steps {
		 script {
		   sh '''
			 docker rmi $DOCKER_IMG || echo "image does not exist"
		   '''
		 }
	  }
     }
	 
  }
}
