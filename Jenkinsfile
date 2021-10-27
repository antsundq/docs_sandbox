pipeline {
	agent any
	stages {
		stage('Clean') {
			steps {
				echo 'Cleaning not configured..'
			}
		}
		
		stage('Fetch') {
			steps {
				dir ('buildsystem'){
					git branch: 'main',
						credentialsId: 'a3b9a927-12e4-4063-a568-c49d0213d2f3',
						url: 'https://github.com/Asteme/Asteme-Buildsystem.git'
				}
			}
		}
		stage('Build') {
			steps {
				echo 'Building not configured..'
			}
		}
		stage('Test') {
			steps {
				echo 'Testing not configured..'
			}
		}
		stage('Deploy') {
			steps {
				echo 'Deploying not configured..'
			}
		}
	}
	options {
		buildDiscarder(logRotator(daysToKeepStr: '3', numToKeepStr: '5'))
	}
		triggers {
		pollSCM('H/5 * * * *')
	}
}