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
		stage('Initialize Build System') {
			steps {
				dir ('buildsystem'){
					git url: 'https://github.com/Astemes/astemes-build-support.git',
						branch: 'main',
						credentialsId: 'Jenkins-Astemes'
					echo 'Build system pulled'
				}
				bat 'git fetch'
				bat "if not exist ${WORKSPACE}\\${REPORT_PATH} mkdir ${WORKSPACE}\\${REPORT_PATH}"
				bat "if not exist ${WORKSPACE}\\${LOG_PATH} mkdir ${WORKSPACE}\\${LOG_PATH}"
			}
		}
		stage('Initialize Python venv') {
			steps {
				dir ('buildsystem'){
					bat 'python -m venv .venv'
					bat '.venv\\scripts\\activate'
					bat 'pip install -r requirements.txt'
				}
				echo 'Python environment initialized'
			}
		}
		stage('Test') {
			steps {
				bat "LabVIEWCLI -OperationName LUnit -ProjectPath \"${WORKSPACE}\\${LV_PROJECT_PATH}\" -TestRunners 8 -ReportPath \"${WORKSPACE}\\${REPORT_PATH}\\lunit.xml\" -ClearIndex TRUE -PortNumber ${LV_PORT_NUMBER} -LogFilePath \"${WORKSPACE}\\${LOG_PATH}\\LabVIEWCLI_LUnit.txt\" -LogToConsole true -Verbosity Default"
			}
		}
		stage('Build') {
			steps {
				bat "LabVIEWCLI -OperationName ExecuteBuildSpec -ProjectPath \"${WORKSPACE}\\${LV_PROJECT_PATH}\" -TargetName \"${LV_TARGET_NAME}\" -BuildSpecName \"${LV_BUILD_SPEC}\" -PortNumber ${LV_PORT_NUMBER} -LogFilePath  \"${WORKSPACE}\\${LOG_PATH}\\LabVIEWCLI_ExecuteBuildSpec.txt\" -LogToConsole true -Verbosity Default"
			}
		}
		stage('Build VIP') {
			steps {
				script{
					echo "START"
					def version = bat(returnStdout: true, script: "@git tag --contains").trim() ? "fix" : "build"
					echo "${version}"
					echo "LabVIEWCLI -OperationName BuildVIP -VIPBPath \"${WORKSPACE}\\${LV_VIPB_PATH}\" -LabVIEWVersion ${LV_VERSION} -IncrementVersion \"${version}\" -PortNumber ${LV_PORT_NUMBER} -LogFilePath \"${WORKSPACE}\\${LOG_PATH}\\LabVIEWCLI_BuildVIP.txt\" -LogToConsole true -Verbosity Default"
					VIP_FILE_PATH = bat(returnStdout: true, script: "LabVIEWCLI -OperationName BuildVIP -VIPBPath \"${WORKSPACE}\\${LV_VIPB_PATH}\" -LabVIEWVersion ${LV_VERSION} -IncrementVersion \"${version}\" -PortNumber ${LV_PORT_NUMBER} -LogFilePath \"${WORKSPACE}\\${LOG_PATH}\\LabVIEWCLI_BuildVIP.txt\" -LogToConsole true -Verbosity Default")
					echo "${VIP_FILE_PATH}"
				}
			}
		}
		stage('Generate Docs'){
			steps{
				dir('buildsystem/mkdocs_builder'){
					bat 'python mkdocs_builder.py --docs_path '+env.WORKSPACE +"\\docs --site_name \"${PROJECT_TITLE}\" --repo_url \"${REPO_URL}\" --author \"${AUTHOR}\" --initial_release ${INITIAL_RELEASE}"
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
			steps{
				bat 'mkdocs gh-deploy --force'
				echo 'Project deployed'
				script{
					def tag = bat(returnStdout: true, script: "git tag --contains").trim()
					def user = "${GITHUB_USER}"
					def repo = "${GITHUB_REPO}"
					def message = bat(returnStdout: true, script: "git tag -n99 -l ${tag}")
					def releaseName = "${RELEASE_TITLE} ${tag}"			
					bat "chmod 777 ./buildsystem/github_release/linux-amd64-github-release"
					def vipPath = VIP_FILE_PATH
					def fileName = "VIPM_Package.txt"
					bat "./buildsystem/github_release/linux-amd64-github-release release --user ${user} --repo ${repo} --tag '${tag}' --name '${releaseName}' --description '${message}' --draft"
					bat "./buildsystem/github_release/linux-amd64-github-release upload --user ${user} --repo ${repo} --tag '${tag}' --name '${fileName}' --file '${vipPath}'"
				}
			}
		}	
	}
	post{
		always{
			echo "${WORKSPACE}\\${REPORT_PATH}\\*.xml"
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