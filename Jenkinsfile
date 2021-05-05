pipeline {
     environment {
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_TAG = "latest"
       STAGING = "aurelien-staging"
       PRODUCTION = "aurelien-production"
       IMAGE_REPO = "guyduche"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG
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
                    curl http://172.17.0.1 | grep -q "Hello world"
                '''
              }
           }
      }
      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                  docker rm -vf ${IMAGE_NAME}
               '''
             }
          }
     }
    stage('Push image on dockerhub') {
           agent any 
           environment {
                DOCKERHUB_LOGIN = credentials('dockerhub_aurelien')
                
            }

           steps {
               script {
                   sh '''
		   docker login --username ${DOCKERHUB_LOGIN_USR} --password ${DOCKERHUB_LOGIN_PSW}
                   docker push ${IMAGE_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                   '''
               }
           }
        }
	    stage('Deploy using ansible') {
		    agent { docker { image 'dirane/docker-ansible:latest' } }
			steps {
			    script {
				    sh ''' 
					    cd ansible
						ansible-playbook -i prod.yml alpinehelloworld.yml
					'''
				}
			}
		}
		stage('test application') {
		    agent { docker { image 'dirane/docker-ansible:latest' } }
			steps {
			    script {
				    sh ''' 
					    cd ansible
						ansible-playbook -i prod.yml test.yml
					'''
				}
			}
		}
	}
}
