pipeline {
	agent any
	tools {
	maven 'mvn'
	}

	stages {
		stage ('hello') {
			stops {
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
