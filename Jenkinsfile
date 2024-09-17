
pipeline{
   agent any 
   tools{
    maven 'M2_HOME'
   }
   stages{
    
    stage("maven"){
        steps{ 
        sh 'mvn clean '
        sh 'mvn validate'
        sh 'mvn compile' 
        sh 'mvn install' 
        sh 'mvn test'
        sh 'mvn package'
        }
    }
   } 
}