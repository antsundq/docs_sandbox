pipeline {
	agent any
	stages {
		stage('Checkout'){
			steps{
				echo "Checkout"
			}
		}
		stage('Initialize Build System') {
			steps {
				dir ('buildsystem'){
					git url: 'https://github.com/Asteme/Asteme-Buildsystem.git',
						branch: 'main',
						credentialsId: 'Jenkins-Asteme'
				}
				echo 'Build system pulled' 
				dir ('buildsystem'){
					bat 'python -m venv .venv'
					bat '.venv\\scripts\\activate'
					bat 'pip install -r requirements.txt'
				}
				echo 'Python environment initialized'
			}
		}
		/*
		stage('Test') {
			steps {
			}
		}
		stage('Build') {
			steps {
				echo 'Building not configured..'
			}
		}
		*/
		stage('Generate Docs'){
			steps{
				dir('buildsystem/mkdocs builder'){
					bat 'python mkdocs_builder.py '+env.WORKSPACE +'\\docs'
				}
				bat 'mkdocs build'
			}
		}
		stage('Deploy') {
			when{
				expression{
					return script {bat(returnStdout: true, script: "@git tag --contains").trim()}
				}
			}
			steps {
				bat 'mkdocs gh-deploy'
				echo 'Project deployed'
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