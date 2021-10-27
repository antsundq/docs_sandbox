pipeline {
	agent any
	stages {
		stage('Clean') {
			steps {
				echo 'Cleaning not configured..'
			}
		}
		
		stage('Fetch Build System') {
			steps {
				dir ('buildsystem'){
					git branch: 'main',
						credentialsId: 'Jenkins-Asteme',
						url: 'https://github.com/Asteme/Asteme-Buildsystem.git'
					sh 'python -m venv .venv'
					sh '.venv\scripts\activate'
					sh 'pip install -r requirements.txt'
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