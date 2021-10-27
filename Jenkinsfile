pipeline {
	agent any
	environment{
		AUTHOR = "Anton Sundqvist"
		INITIAL_RELEASE = 2021
	}
	stages {
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
					bat 'python mkdocs_builder.py --docs_path '+env.WORKSPACE +"\\docs --author '${AUTHOR}' --initial_release ${INITIAL_RELEASE}"
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