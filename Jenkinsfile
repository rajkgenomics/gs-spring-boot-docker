node('master'){
   def mvnHome
   stage('Preparation') { // for display purposes
      // Get some code from this GitHub repository
      checkout scm
 
      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'M3'
   }
   stage('Test before') {
           // some block
           sh "oc project rajtest"
           sh "oc get bc"
           sh 'ls -l'
           sh "oc start-build hellodocker2 --from-dir=. --follow"
       }
   stage('Build') {
      // Run the maven build
      if (isUnix()) {
          dir('complete') {
           // some block
           sh "'${mvnHome}/bin/mvn' -DskipTests clean package"
          }
      } else {
         bat(/"${mvnHome}\bin\mvn" -DskipTests clean package/)
      }
   }
   stage('Setup1') {
           // some block
           sh "oc project rajtest"
           sh "oc get bc"
           sh 'ls -l'
           sh "oc start-build hellodocker2 --from-dir=. --follow"
       }
   stage('Setup2') {
       dir('complete/target') {
           // some block
           sh 'pwd'
           sh 'ls -l'
       }
   }
}




