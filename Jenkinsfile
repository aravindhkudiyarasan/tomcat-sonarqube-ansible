pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    options {
        buildDiscarder logRotator(daysToKeepStr: '5', numToKeepStr: '7')
    }
    //stages{
      //  stage('Build'){
        //    steps{
          //       sh script: 'mvn clean package'
            //     archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true
            //}
        //}
		stage('sonaqube analysis'){
		    steps{
			    withSonarQubeEnv('sonarqube'){
				sh 'mvn clean package sonar:sonar'
				}
			}
		}
		stage("Quality Gate"){
		    timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qg.status != 'OK') {
             error "Pipeline aborted due to quality gate failure: ${qg.status}"
			}
		}
	}
        stage('Upload War To Nexus'){
            steps{
                script{

                    def mavenPom = readMavenPom file: 'pom.xml'
                    def nexusRepoName = mavenPom.version.endsWith("SNAPSHOT") ? "simpleapp-snapshot" : "simpleapp-release"
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'simple-app', 
                            classifier: '', 
                            file: "target/simple-app-${mavenPom.version}.war", 
                            type: 'war'
                        ]
                    ], 
                    credentialsId: 'nexus3', 
                    groupId: 'in.javahome', 
                    nexusUrl: '192.168.1.5:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: nexusRepoName, 
                    version: "${mavenPom.version}"
                    }
            }
        }
	}	

