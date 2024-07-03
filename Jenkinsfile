# syntax=docker/dockerfile:1

FROM python:3.10.12-slim-bullseye
WORKDIR /python-docker

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0"]
2. I’ve uploaded this code to a GITHUB repo.

3. In Jenkins I have written my declarative syntax, consisting of the following:

Comments have been added to add explanation to my code:

pipeline {
    
    // run on any available agent in the jenkins environment
    agent any 

    //set environment variables
    environment {

        //docker hub
        registry = "dineshdadisetty/flaskapp"

        // jenkins credential ID
        registryCredential = 'dineshdadisetty'
        dockerImage = ''
    }
    
    // Retreive code from Github 
    stages {
        stage('Cloning Git') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github_user', url: 'https://github.com/clinton-pillay7/flask-docker']])       
            }
        }
    
    // Building Docker images
    stage('Building image') {
      steps{
        script {
            dockerImage = docker.build registry
        }
      }
    }
     // Uploading Docker images into Docker Hub
    stage('Upload Image') {
     steps{    
         script {
            docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
            }
        }
      }
    }
     // Stopping Docker containers for cleaner Docker run
     stage('docker stop container') {
         steps {
            sh 'docker ps -f name=mypythonappContainer -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -fname=mypythonappContainer -q | xargs -r docker container rm'
         }
       }
    
    
    // Running Docker container, make sure port 5000 is opened in 
    stage('Docker Run') {
     steps{
         script {
            dockerImage.run("-p 5000:5000 --rm --name mypythonappContainer")
         }
      }
    }


    //Asking user to approve via email, or via the jenkins console interface, 
    //both are in parallel, So the user can try either approval method
    stage("Approve test") {
            parallel {
                stage('Approve via link') {
                    steps{
                        script {
                                env.RELEASE_SCOPE = input message: 'User input required', ok: 'Release!',
                                parameters: [choice(name: 'RELEASE_SCOPE', choices: ["accept", "reject"], description: 'Can we release?')]
                                        }
                                    }
                            }
                stage('Approve via email') {
                    steps {
                        emailext to: 'nose-voice@slmhfv4o.mailosaur.net', subject: 'New build is waiting for your decision', body: '<!DOCTYPE html><html><p>Hi, A new build has been deployed, please view it by visiting <a href=http://192.168.8.114:5000/>"http://192.168.8.114:5000/"</a>, please approve this deployment by clicking on <a href=http://192.168.8.114:8080/job/flask-docker/${BUILD_NUMBER}/input/>"http://192.168.8.114:5000/"</a></p></html>', attachLog: true
                        timeout(time: 60, unit: 'MINUTES') {
            }
        }
                }
            }
        }

    // Deployment is done if the apporval is accepted, by editing the nginx.conf file, to add this docker container to the load balancer pool.     
    stage("Deploy"){
        steps {
            script{
                if (env.RELEASE_SCOPE == "accept"){
                    script{
                        sh "sudo chmod 777 script.sh"
                        sh "./script.sh"
                    }
                }
                else {
                    sh "echo 'deploy refused'"
                }
            }
                
           }
        
    }

  }
}
