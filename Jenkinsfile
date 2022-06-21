pipeline {
  agent { 
    label 'docker' 
  }

  options {
    skipStagesAfterUnstable()
  }
  
  environment {
    TAG_NAME = env.BRANCH_NAME.replace('/','_')
    DOCKER_IMAGE = "flight_review:${TAG_NAME}"
    REGISTRY = "https://051301119471.dkr.ecr.us-east-1.amazonaws.com"
    CRED_ID = "AWS_ECR"
  }
  
  stages {
    stage('Build') {
      steps {   
        echo "Building docker image from ${env.BRANCH_NAME}"          

        // Just to see what's in the environment here.
        sh 'printenv'

        // Pull submodules. Pipeline doesn't do that for some reason
        sh 'git submodule update --init --recursive'

        script {
          
        // Store the current git commit rev in a variable for later use.
          GIT_COMMIT_REV = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
          // Add a file in the build products indicating that we built it.
          writeFile file: "${WORKSPACE}/version.json", text: "{ \"build\": ${env.BUILD_ID}, \"commit\": \"${GIT_COMMIT_REV}\", \"datetime\": \"${java.time.LocalDateTime.now()}\" }"
          // Inject it into the docker image
          sh 'cat ${WORKSPACE}/version.json'
        }
        dir("app") {
        // Build the docker image. The image is our artifact.        
          script {
            targetImage = docker.build("${env.DOCKER_IMAGE}",  '-f ./Dockerfile .')
          }        
        }
      }
    }
    
    stage('Publish') {
      steps {
        echo 'Publishing docker image'

        // push the docker image to the registry.
        script {
          docker.withRegistry("${env.REGISTRY}", "ecr:us-east-1:${env.CRED_ID}") {
            targetImage.push()
          }
        }
      }
    }
  }
}
