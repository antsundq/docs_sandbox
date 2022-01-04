pipeline {
	agent any
	environment{
		PROJECT_TITLE = "Docs Sandbox"
		REPO_URL = "https://github.com/sunqn/docs_sandbox"
		AUTHOR = "Anton Sundqvist"
		INITIAL_RELEASE = 2021
		RELEASE_TITLE = "Release"
	}
	stages {
		stage('Initialize Build System') {
			steps {
				dir ('buildsystem'){
					git url: 'https://github.com/Astemes/astemes-buildsystem.git',
						branch: 'main',
						credentialsId: 'Jenkins-Astemes'
					echo 'Build system pulled'
				}
				sh 'git fetch'
			}
		}
		/*
		stage('Initialize Python venv') {
			steps {
				dir ('buildsystem'){
					sh 'python3 -m venv .venv'
					sh '.venv\\scripts\\activate'
					sh 'pip install -r requirements.txt'
				}
				echo 'Python environment initialized'
			}
		}		
		stage('Test') {
			steps {
			}
		}
		stage('Build') {
			steps {
				echo 'Building not configured..'
			}
		}*/
		stage('Build VIP') {
			steps {
				echo 'Building not configured..'
				script{
					VIP_FILE_PATH = "path/vip"
				}
			}
		}
		/*
		stage('Publish Release') {
			steps {
				echo 'Building not configured..'
			}
		}
		stage('Generate Docs'){
			steps{
				dir('buildsystem/mkdocs_builder'){
					sh 'python mkdocs_builder.py --docs_path '+env.WORKSPACE +"\\docs --site_name \"${PROJECT_TITLE}\" --repo_url \"${REPO_URL}\" --author \"${AUTHOR}\" --initial_release ${INITIAL_RELEASE}"
				}
				sh 'mkdocs build'
			}
		}
		stage('Deploy') {
			when{
				expression{
					return script {bat(returnStdout: true, script: "@git tag --contains").trim()}
				}
			}
			steps {
				sh 'mkdocs gh-deploy --force'
				echo 'Project deployed'
			}
		}
		*/	
		stage('Release') {
			steps{
				dir ('buildsystem'){
					script{
						def tag = sh(returnStdout: true, script: "git tag --contains").trim()
						def message = sh(returnStdout: true, script: "git tag -n99 -l ${tag}")
						def releaseName = "${RELEASE_TITLE} ${tag}"			
						sh "chmod 777 linux-amd64-github-release"
						def releaseInfo = sh(returnStdout: true, script: "./github_release/linux-amd64-github-release -h")
						def vipPath = VIP_FILE_PATH
						echo releaseInfo
					}
				}
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