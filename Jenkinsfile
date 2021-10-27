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
						credentialsId: 'Jenkins-Asteme',
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
		stage('Generate Docs'){
			steps{
				echo 'Docs not configured..'
				dir('buildsystem/mkdocs builder'){
					sh 'python mkdocs_builder.py'
				}
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