pipeline {
	agent any
	environment{
		PROJECT_TITLE = "Docs Sandbox"
		REPO_URL = "https://github.com/sunqn/docs_sandbox"
		REPORT_PATH = "reports"
		AUTHOR = "Anton Sundqvist"
		INITIAL_RELEASE = 2021
		LV_PROJECT_PATH = "source\\test.lvproj"
		LV_VIPB_PATH = "source\\test.vipb"
		LV_BUILD_SPEC = "Demo"
		LV_VERSION = "20.0"
		COMMIT_TAG = "${bat(returnStdout: true, script: '@git tag --contains').trim()}"
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
				dir("build_support"){
					pullBuildSupport()
					initPythonVenv "requirements.txt"
					buildDocs "${PROJECT_TITLE}", "${REPO_URL}", "${AUTHOR}", "${INITIAL_RELEASE}"
				}
			}
		}
		stage('Deploy') {
			when{
				expression{
					return bat(returnStdout: true, script: "@git tag --contains").trim()
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