node('master'){
    def mvnHome
    // The artifactory-prod one is the one that has the credentials to allow more access to deploy etc
    def server = Artifactory.server('artifactory-prod')
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo

    stage ('Clone') {
        checkout scm
    }

    stage ('Artifactory configuration') {
        // Obtain an Artifactory server instance, defined in Jenkins --> Manage:
        rtMaven = Artifactory.newMavenBuild()
        rtMaven.tool = 'M3' // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server

        rtMaven.deployer.artifactDeploymentPatterns.addInclude("*.jar").addExclude("*.placeholder")
       
        rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run

        buildInfo = Artifactory.newBuildInfo()
    }
 
    stage ('Test') {
        // rtMaven.run pom: 'complete/pom.xml', goals: 'clean test'
    }
        
    stage ('Package') {
        rtMaven.run pom: 'complete/pom.xml', goals: 'clean package -DskipTests', buildInfo: buildInfo
    }
 
    stage ('Deploy') {
        rtMaven.deployer.deployArtifacts buildInfo
    }
        
    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }

    stage('Test before') {
/** The logical name references a Jenkins cluster configuration which implies **/
/** API Server URL, default credentials, and a default project to use within the closure body. **/
openshift.withCluster( 'OpenshiftNonProdLoggingInAsAdmin' ) {
    openshift.withProject( 'rajtest' ) {
        echo "Now inside the openshift project: ${openshift.project()}"
        sh 'pwd'
        sh 'ls -l'
        openshift.selector("bc","hellodocker2").startBuild("--from-dir=.","--follow")
    }
}
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




