
pipeline{
   agent any 
   environment {
    BRANCH_NAME = 'main'
    GIT_URL = 'https://github.com/Minnie2629/geo.git'
    GITHUB_CREDENTIALS = 'github-credentials'  
    
   }
   stages{
    
    stage('checkout'){
        steps{ 
        git branch: "${BRANCH_NAME}", credentialsId: "${GITHUB_CREDENTIALS}",\
         url: "${GIT_URL}"
        }
    }
   } 
}