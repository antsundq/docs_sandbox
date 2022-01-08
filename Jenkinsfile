pipeline {
	agent any
	environment{
		PROJECT_TITLE = "Docs Sandbox"
		REPORT_PATH = "reports"
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
				//Execute LabVIEW build spec
				buildLVBuildSpec "${WORKSPACE}\\${LV_PROJECT_PATH}", "${LV_BUILD_SPEC}"
				//Build VIPM package
				script{VIP_FILE_PATH = buildVIPackage "${WORKSPACE}\\${LV_VIPB_PATH}", "${LV_VERSION}"}
				//Build mkdocs documentation
				pullBuildSupport()
				initPythonVenv "requirements.txt"
				buildDocs "${PROJECT_TITLE}", "${REPO_URL}", "${AUTHOR}", "${INITIAL_RELEASE}"
			}
		}
		stage('Deploy') {
			when{
				expression{
					return script {bat(returnStdout: true, script: "@git tag --contains").trim()}
				}
			}
			steps{
				deployGithubPages()
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