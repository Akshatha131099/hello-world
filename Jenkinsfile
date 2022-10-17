pipeline {
	agent any
	tools {
	maven 'mvn'
	}

	stages {
		stages ('Hello') {
			steps {
				git 'https://github.com/Akshatha131099/hello-world.git'
			}
		}
		stage('compile') {
			steps {
				sh 'mvn clean package'
			}
		}
		stage('deployment') {
			steps {
				sh 'mvn deploy'
			}
		}
		stage('docker') {
			steps {
				sh 'mvn docker'
			}
		}
	}
}
