pipeline {
	agent any
	tools {
	maven 'mvn'
	}

	stages {
		stage ('Hello') {
			steps {
				git 'https://github.com/Akshatha131099/hello-world.git'
			}
		}
		stage('compile') {
			steps {
				echo 'mvn clean package'
			}
		}
		stage('deployment') {
			steps {
				echo 'mvn deploy'
			}
		}
		stage('docker') {
			steps {
				echo 'mvn docker'
			}
		}
		stage('ansible') {
			steps {
				echo 'mvn ansible'
			}
		}
	}
}
