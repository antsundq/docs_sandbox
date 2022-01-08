pipeline {
	agent any
	environment{
		PROJECT_TITLE = "Docs Sandbox"
		REPORT_PATH = "reports"
		LOG_PATH = "logs" //to be deleted
		REPO_URL = "https://github.com/sunqn/docs_sandbox"
		GITHUB_USER = "sunqn"
		GITHUB_REPO = "docs_sandbox"
		AUTHOR = "Anton Sundqvist"
		INITIAL_RELEASE = 2021
		RELEASE_TITLE = "Release"
		GITHUB_TOKEN = credentials('github-token')
		LV_PROJECT_PATH = "source\\test.lvproj"
		LV_VIPB_PATH = "source\\test.vipb"
		LV_BUILD_SPEC = "Demo"
		LV_TARGET_NAME = "My Computer"//to be deleted
		LV_PORT_NUMBER = 3363 //to be deleted
		LV_VERSION = "20.0"
	}
	stages {
		stage('Initialize') {
			steps {
				library 'astemes-build-support'
				initWorkspace()
			}
		}
		stage('Test') {
			steps {
				runLUnit "${WORKSPACE}\\${LV_PROJECT_PATH}", "${WORKSPACE}\\${REPORT_PATH}"
			}
		}
		stage('Build') {
			steps {
				buildLVBuildSpec "${WORKSPACE}\\${LV_PROJECT_PATH}", "${LV_BUILD_SPEC}"
				script{
					VIP_FILE_PATH = buildVIPackage "${WORKSPACE}\\${LV_VIPB_PATH}", "${LV_VERSION}"
				}
				echo "Built package: ${VIP_FILE_PATH}"
				
				dir ('buildsystem'){
					git url: 'https://github.com/Astemes/astemes-build-support.git',
						branch: 'main',
						credentialsId: 'Jenkins-Astemes'
					bat 'git fetch'
					initPythonVenv "requirements.txt"
				}
				dir('buildsystem/mkdocs_builder'){
					bat 'python mkdocs_builder.py --docs_path '+env.WORKSPACE +"\\docs --site_name \"${PROJECT_TITLE}\" --repo_url \"${REPO_URL}\" --author \"${AUTHOR}\" --initial_release ${INITIAL_RELEASE}"
				}
				bat 'python -m mkdocs build'
			}
		}
		stage('Deploy') {
			when{
				expression{
					return script {bat(returnStdout: true, script: "@git tag --contains").trim()}
				}
			}
			steps{
				bat 'python -m mkdocs gh-deploy --force'
				echo 'Documentation deployed'
				script{
					def tag = "v1.0.10" 
					//bat(returnStdout: true, script: "@git tag --contains").trim()
					def user = "${GITHUB_USER}"
					def repo = "${GITHUB_REPO}"
					def message = bat(returnStdout: true, script: "@git tag -n99 -l ${tag}").replaceAll("[\\n]", "")
					def releaseName = "${RELEASE_TITLE} ${tag}"
					//sh "chmod 777 ./buildsystem/github_release/linux-amd64-github-release"
					def vipPath = VIP_FILE_PATH
					File f = new File(vipPath)
					fileName = f.getName()
					//Create Release
					bat "buildsystem\\github_release\\github-release.exe release --user ${user} --repo ${repo} --tag \"${tag}\" --name \"${releaseName}\" --description \"${message}\" --draft"
					//Upload VIP file
					bat "buildsystem\\github_release\\github-release.exe upload --user ${user} --repo ${repo} --tag \"${tag}\" --name \"${fileName}\" --file \"${vipPath}\""
				}
			}
		}
	}
	post{
		always{
			junit "${REPORT_PATH}\\*.xml"
		}
	}
	options {
		buildDiscarder(logRotator(daysToKeepStr: '3', numToKeepStr: '5'))
	}
		triggers {
		pollSCM('H/5 * * * *')
	}
}