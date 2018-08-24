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
 
    stage ('UnitTest') {
        // rtMaven.run pom: 'complete/pom.xml', goals: 'clean test'
    }
        
    stage ('Package') {
        rtMaven.run pom: 'complete/pom.xml', goals: 'clean package -DskipTests', buildInfo: buildInfo
    }
 
    stage ('DeployAppArtefact') {
        rtMaven.deployer.deployArtifacts buildInfo
    }
        
    stage ('PublishBuildInfo') {
        server.publishBuildInfo buildInfo
    }
    stage('Delete') {
      script {
          /** The logical name references a Jenkins cluster configuration which implies **/
          /** API Server URL, default credentials, and a default project to use within the closure body. **/
          openshift.withCluster( 'OpenshiftProdLoggingInAsAdmin' ) {
                echo "NOTE : A Pre-Requisite for this pipeline is that project rajtest exists and has been created"
                openshift.withProject( 'rajtest' ) {
                      echo "Now inside the openshift project: ${openshift.project()}"
 
                      if (openshift.selector('dc','hellodocker2').exists()) {
                         echo "Now Deleting DC, SVC, and Route objects..."
                         openshift.selector('dc','hellodocker2').delete()
                         openshift.selector('svc','hellodocker2').delete()
                         openshift.selector('route','hellodocker2').delete()
                      }
                      if (openshift.selector('bc','hellodocker2').exists()) {
                         echo "Now Deleting BC object..."
                         openshift.selector('bc','hellodocker2').delete()
                      }
                }
          }
      }
    }
    stage('CreateTemplates') {
      script {
          /** The logical name references a Jenkins cluster configuration which implies **/
          /** API Server URL, default credentials, and a default project to use within the closure body. **/
          openshift.withCluster( 'OpenshiftProdLoggingInAsAdmin' ) {
                echo "NOTE : A Pre-Requisite for this pipeline is that project rajtest exists and has been created"
                openshift.withProject( 'rajtest' ) {
                      echo "Now inside the openshift project: ${openshift.project()}"

                      def models = openshift.process("--filename=/var/lib/jenkins/jobs/testing123/workspace/complete/openshift/deploy-template5.yml")

                      // A list of Groovy object models that were defined in the template will be returned.
                      echo "Creating this template will instantiate ${models.size()} objects"

                      // You can pass this list of object models directly to the create API
                      def created = openshift.apply( models )
                      echo "Template Created: ${created.names()}"

                      def models2 = openshift.process("--filename=/var/lib/jenkins/jobs/testing123/workspace/complete/openshift/build-template5.yml")

                      // A list of Groovy object models that were defined in the template will be returned.
                      echo "Creating this template will instantiate ${models.size()} objects"

                      // You can pass this list of object models directly to the create API
                      def created2 = openshift.apply( models2 )
                      echo "Template Created: ${created2.names()}"
                }
          }
      }
    }
    stage('BuildImageToArtifactory') {
      script {
          /** The logical name references a Jenkins cluster configuration which implies **/
          /** API Server URL, default credentials, and a default project to use within the closure body. **/
          openshift.withCluster( 'OpenshiftProdLoggingInAsAdmin' ) {
                openshift.withProject( 'rajtest' ) {
                      echo "Now inside the openshift project: ${openshift.project()}"
                      openshift.selector("bc","hellodocker2").startBuild("--from-dir=/var/lib/jenkins/jobs/testing123/workspace","--wait","--follow")
                }
          }
      }
    }
    stage('PullFromArtifactory') {
      script {
          /** The logical name references a Jenkins cluster configuration which implies **/
          /** API Server URL, default credentials, and a default project to use within the closure body. **/
          openshift.withCluster( 'OpenshiftProdLoggingInAsAdmin' ) {
                echo "NOTE : A Pre-Requisite for this pipeline is that project rajtest exists and has been created"
                openshift.withProject( 'rajtest' ) {
                      echo "Now inside the openshift project: ${openshift.project()}"

                      def models3 = openshift.process("--filename=/var/lib/jenkins/jobs/testing123/workspace/complete/openshift/pull-template5.yml")

                      // A list of Groovy object models that were defined in the template will be returned.
                      echo "Creating this template will instantiate ${models3.size()} objects"

                      // You can pass this list of object models directly to the create API
                      def created3 = openshift.apply( models3 )
                      echo "Template Created: ${created3.names()}"
                }
          }
      }
    }
    stage('DeployImageInOpenshift') {
/*
      script {
          /** The logical name references a Jenkins cluster configuration which implies **/
          /** API Server URL, default credentials, and a default project to use within the closure body. **/
          openshift.withCluster( 'OpenshiftProdLoggingInAsAdmin' ) {
                openshift.withProject( 'rajtest' ) {
                      echo "Now inside the openshift project: ${openshift.project()}"

                      echo "Now Trying to deploy..."
                      openshift.selector("dc","hellodocker2").rollout().latest()

                      def latestDeploymentVersion = openshift.selector('dc',"hellodocker2").object().status.latestVersion
                      echo "Checking the deployment deploy...${latestDeploymentVersion}"
                      def rc = openshift.selector('rc', "hellodocker2-${latestDeploymentVersion}")
                      rc.untilEach(1){
                      def rcMap = it.object()
                              return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
                      }
                }
          }
      } */
    }
    stage('Testing') {
      script {
          echo "Run some smoke tests here on the Pod you just deployed...Test all ok..."
          echo "Tests to be decided and added....."
     }
    }
    stage('Promote-And-Tag') {
      script {
          echo "We can either promote within the Openshift Docker registry or the Artifactory"
          echo "Use the artifactory plugin to do this....TODO"
     }
    }
}
