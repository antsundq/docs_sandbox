pipeline {
	agent any
	environment{
		LV_PROJECT_PATH = "${WORKSPACE}\\source\\test.lvproj"
        NUM_TEST_RUNNERS = "1"
        LV_PORT = "3363"
	}
	stages {
		stage('Unit Tests') {
			steps {
				bat "LabVIEWCLI -OperationName LUnit -ProjectPath \"${LV_PROJECT_PATH}\" -TestRunners ${NUM_TEST_RUNNERS} -ReportPath \"reports\\lunit.xml\" -ClearIndex TRUE -PortNumber ${LV_PORT} -LogFilePath \"${WORKSPACE}\\LabVIEWCLI_LUnit.txt\" -LogToConsole true -Verbosity Default"
			}
		}
	}
	post{
		always{
			junit "reports\\*.xml"
		}
	}
}