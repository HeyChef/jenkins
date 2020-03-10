// Lien vers Nexus, doit correspondre à l'instance paramétrée dans Jenkins
def nexusId = 'localnexus'

/* *** Configuration de Nexus pour Maven ***/
// URL de Nexus
def nexusUrl = 'http://172.18.0.4:8081'

// Repo Id (provient du settings.xml nexus pour récupérer user/password)
def mavenRepoId = 'nexusLocal'

/* *** Repositories Nexus *** */
def nexusRepoSnapshot = "maven-snapshots"
def nexusRepoRelease = "maven-releases"



/* *** Détail du projet, récupéré dans le pipeline en lisant le pom.xml *** */
def groupId = ''
def artefactId = ''
def filePath = ''
def packaging = ''
def version = ''

// Variable utilisée pour savoir si c'est une RELEASE ou une SNAPSHOT
def isSnapshot = true

pipeline {
   agent any

   stages {
      stage('Checkout') {
         steps {
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/HeyChef/jenkins.git']]])
         }
      }
      stage('Get info from POM') {
          steps {
            script {
                pom = readMavenPom file: 'pom.xml'
                groupId = pom.groupId
                artifactId = pom.artifactId
                packaging = pom.packaging
                version = pom.version
                filepath = "target/${artifactId}-${version}.jar"
                isSnapshot = version.endsWith("-SNAPSHOT")
            }
            echo groupId
            echo artifactId
            echo packaging
            echo version
            echo filepath
            echo "isSnapshot: ${isSnapshot}"
          }
      }
      stage('Build') {
          steps {
              sh 'mvn clean package'
          }
      }

      stage('Checkstyle') {
               steps{
      				sh 'mvn checkstyle:checkstyle'
                   }
      }

      stage('Spotbugs') {
       steps{
                sh 'mvn spotbugs:spotbugs'
            }
      }
      stage('Push RELEASE to Nexus') {
          when { expression { !isSnapshot } }
          steps {
            nexusPublisher nexusInstanceId: 'localnexus', nexusRepositoryId: "${nexusRepoRelease}", packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: "${filepath}"]], mavenCoordinate: [artifactId: "${artifactId}", groupId: "${groupId}", packaging: "${packaging}", version: "${version}"]]]
          }
      }
   }
/*
   post{
      always{
        recordIssues enabledForFailure : true, tools: [mavenConsole(),java(),javaDoc()]*/
       // junit '**/surefire-reports/*.xml'
       /* recordIssues enabledForFailure : true, tool: checkStyle()
        recordIssues enabledForFailure : true, tool: spotBugs()*/
        //recordIssues enabledForFailure : true, tool: cpd(pattern:'**/target/cpd.xml')
        //recordIssues enabledForFailure : true, tool: pmdParser(pattern:'**/target/pmd.xml')
   /*
      }
   }*/
}
