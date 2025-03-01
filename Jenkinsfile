
pipeline {
    agent any
    tools {
        maven "M3"
        // sonarQubeScanner 'SonarQube-Server'
    }

	environment {
        pom = readMavenPom(file: 'calculator/pom.xml')
        artifactId = pom.getArtifactId()
        version = pom.getVersion()
        name = pom.getName()
        groupId = pom.getGroupId()
	}

    stages {
        stage('Check Git pull') {
            steps {
                sh 'ls -l'
				sh 'rm -rf /root/.m2/repository/*'   
            }
        }

		stage('Build') {
		steps {
			dir('calculator') {
			sh 'cat src/main/java/com/echios/App.java '
			sh 'mvn clean install'
			}
		}
		}

		stage('Environment details') {
		steps {
			dir('calculator') {
			script {
				echo "Version: ${version}"
				echo "Group ID: ${groupId}"
				echo "Artifact ID: ${artifactId}"
			}
			}
		}
		}

        stage('SonarQube Analysis...') {
            steps {
                dir('calculator') {
                    script {
                        withSonarQubeEnv(credentialsId: 'sonar-scan') {
                        sh 'mvn clean package sonar:sonar -Dsonar.coverage.exclusions=**/*'
                        }
                    }
                }
            }
        }

        stage('Send to Nexus snapshot repo') {
            steps {
                dir('calculator') {
                echo 'Nexus upload'
                nexusArtifactUploader(
                    credentialsId: 'nexus-jenkins',
                    groupId: "${groupId}",
                    nexusUrl: 'animated-space-halibut-7xx47q6xxgrh4p9-8081.app.github.dev',
                    nexusVersion: 'nexus3',
                    protocol: 'https',
                    repository: 'echop',
                    version: "${version}",
                    artifacts: [
                        [
                            artifactId: "${artifactId}",
                            classifier: '',
                            file: "target/${artifactId}-${version}.jar",
                            type: 'jar'
                        ]
                    ]
                )
                }
            }
        }

stage('JiraUpdate') {
    steps {
        jiraComment body: "Sonar Passed & pushed to Nexus (Job: ${env.JOB_NAME}, Build: ${env.BUILD_NUMBER})", issueKey: 'MBA-3'
    }
}

// stages finish
    }
}
