pipeline {
	agent any
	environment{
		PROJECT_TITLE = "Docs Sandbox"
		REPO_URL = "https://github.com/antsundq/docs_sandbox"
		AUTHOR = "Anton Sundqvist"
		INITIAL_RELEASE = 2021
		LV_PROJECT_PATH = "source\\test.lvproj"
		LV_BUILD_SPEC = "Demo"
		LV_VIPB_PATH = "source\\test.vipb"
		LV_VERSION = "20.0"
		COMMIT_TAG = "${bat(returnStdout: true, script: '@git fetch & git tag --contains').trim()}"
	}
	stages {
		stage('Initialize') {
			steps {
				echo "Commit Tag: ${COMMIT_TAG}"
				library 'astemes-build-support'
				initWorkspace()
				dir("build_support"){
					pullBuildSupport()
					initPythonVenv "requirements.txt"
				}
			}
		}
		stage('Test') {
			steps {
				runLUnit "${LV_PROJECT_PATH}"
			}
		}
		stage('Build') {
			steps {
				//Execute LabVIEW build spec
				buildLVBuildSpec "${LV_PROJECT_PATH}", "${LV_BUILD_SPEC}"
				
				//Build VIPM package
				script{VIP_FILE_PATH = buildVIPackage "${LV_VIPB_PATH}", "${LV_VERSION}"}
				
				//Build mkdocs documentation
				buildDocs "${PROJECT_TITLE}", "${REPO_URL}", "${AUTHOR}", "${INITIAL_RELEASE}"
			}
		}
		stage('Deploy') {
			when{
				expression{
					"${COMMIT_TAG}"
				}
			}
			environment{
				GITHUB_TOKEN = credentials('github-token')
			}
			steps{
				deployGithubPages()
				deployGithubRelease "${REPO_URL}", "${COMMIT_TAG}", "${VIP_FILE_PATH}"
			}
		}
	}
	post{
		always{
			junit "reports\\*.xml"
		}
	}
	options {
		buildDiscarder(logRotator(daysToKeepStr: '3', numToKeepStr: '5'))
	}
		triggers {
		pollSCM('H/5 * * * *')
	}
}