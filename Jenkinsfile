pipeline {
	agent any
	environment{
		PROJECT_TITLE = "Docs Sandbox"
		REPORT_PATH = "reports"
		LOG_PATH = "logs"
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
		LV_TARGET_NAME = "My Computer"
		LV_PORT_NUMBER = 3363
		LV_VERSION = "20.0"
	}
	stages {
		stage('Initialize') {
			steps {
				initWorkspace()
				initPythonVenv "requirements.txt"
			}
		}
		stage('Test') {
			steps {
				runLUnit "${WORKSPACE}\\${LV_PROJECT_PATH}", "${WORKSPACE}\\${REPORT_PATH}"
				
			}
		}
		stage('Build') {
			steps {
				bat "LabVIEWCLI -OperationName ExecuteBuildSpec -ProjectPath \"${WORKSPACE}\\${LV_PROJECT_PATH}\" -TargetName \"${LV_TARGET_NAME}\" -BuildSpecName \"${LV_BUILD_SPEC}\" -PortNumber ${LV_PORT_NUMBER} -LogFilePath  \"${WORKSPACE}\\${LOG_PATH}\\LabVIEWCLI_ExecuteBuildSpec.txt\" -LogToConsole true -Verbosity Default"

				script{
					try{
						def version = bat(returnStdout: true, script: "@git tag --contains").trim() ? "fix" : "build"
						String rawOut = bat(returnStdout: true, script: "@LabVIEWCLI -OperationName BuildVIP -VIPBPath \"${WORKSPACE}\\${LV_VIPB_PATH}\" -LabVIEWVersion ${LV_VERSION} -IncrementVersion \"${version}\" -PortNumber ${LV_PORT_NUMBER} -LogFilePath \"${WORKSPACE}\\${LOG_PATH}\\LabVIEWCLI_BuildVIP.txt\" -LogToConsole true -Verbosity Default")
						def index = rawOut.indexOf('Operation output:')
						String buildOut = rawOut.substring(index)
						String[] buildOutRows = buildOut.split("\\n")
						VIP_FILE_PATH = buildOutRows[1].trim()
						echo "VIP_FILE_PATH: \n${VIP_FILE_PATH}"
					}
					catch (err){
						echo "${err}"
					}
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