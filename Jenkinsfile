pipeline {
    agent any
	 tools{
	  maven 'maven3'
	  }
    stages {
        
		stage('SCM') {
            steps {
                git url: 'https://github.com/aravindhkudiyarasan/sample-app.git'
            }
        }
        stage('Build && SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    
                        sh 'mvn clean package sonar:sonar'
                    
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 30, unit: 'SECONDS') {
				    script{
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                              error "Pipeline aborted due to quality gate failure: ${qg.status}"
                          }
                }    }
            }
        }
        stage("MAVEN BUILD") {
            steps{
                sh 'mvn clean package'
        }   }
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
}
