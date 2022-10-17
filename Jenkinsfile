import com.cloudbees.groovy.cps.NonCPS

def FAILED_STAGE = 'UNKNOWN'
def commitMsgExtract = 'UNKNOWN'
def gitTag = ''
def force_qac = false
	pipeline {
		agent {
			node {
                label 'L2H4060'
            }
		}
		parameters {
			booleanParam(name: 'qacCheckAndQAVDashboard',
                defaultValue: false,)
			booleanParam(name: 'ReleaseNotes',
                defaultValue: false,)	
			string(name: 'PreviousTagName', defaultValue: 'L2H4060_22.17.00.00_PWB1', description: 'Previous Release Notes Tag from GitHub',)
        	string(name: 'LatestTagName', defaultValue: 'L2H4060_Release_Note', description: 'Latest Release Notes Tag from GitHub',)
        	string(name: 'ReleaseVersion', defaultValue: 'TestRelease', description: 'Latest Release Version',)									
		}
		environment {
			PYTHONPATH = 'C:/prjtools/automationBookshelf/2020_Bookshelf_1211'
			PAGEPYTHONPATH = 'C:/prjtools/python/ver_3.7.6_p3'
			GIT_PYTHON_GIT_EXECUTABLE = 'C:/prjtools/git/ver_1.4/bin/git.exe'
			PY_EXE = "C:/prjtools/python/ver_3.7.6_p3/python.exe"
			PATH = "C:/prjtools/python/ver_3.7.6_p3;C:/prjtools/python/ver_3.7.6_p3/Scripts;C:/prjtools/mingw32/ver_3.82.90/bin;C:/prjtools/git/ver_1.4/bin/;$PATH"
     	//	CURL_EXE = 'C:/prjtools/git/ver_1.4/mingw64/bin/curl.exe'
			EMAIL_ID = "bhavya.r@partner.magna.com; Devaraju.DG@partner.magna.com; gopinath.ramanathan@magna.com;"
		}
		options {
			timestamps()
			timeout(time: 4, unit: 'HOURS')
			disableConcurrentBuilds()
		}
        stages {
		/******************************************************************************************************
		*      ______  __    __   _______   ______  __  ___
		*     /      ||  |  |  | |   ____| /      ||  |/  /
		*    |  ,----'|  |__|  | |  |__   |  ,----'|  '  /
		*    |  |     |   __   | |   __|  |  |     |    <
		*    |  `----.|  |  |  | |  |____ |  `----.|  .  \
		*     \______||__|  |__| |_______| \______||__|\__\
		*
		******************************************************************************************************/
			stage('checkBuildEnvironment'){
				environment {
					PYTHONPATH = "C:/prjtools/automationBookshelf/2020_Bookshelf_1211"
					PACKAGER   = "package_manager/packager.py"
				}
                steps {
                    script {
				 // Modify Job Description
					echo "******** Get email Id of commit author ********"
					env.COMMITTER_EMAIL = bat (script: "@${GIT_PYTHON_GIT_EXECUTABLE} show -s --format='%%ae' ${GIT_COMMIT}", returnStdout: true).replace("\'","")
					print(env.COMMITTER_EMAIL)											
				    def shortGitCommit = env.GIT_COMMIT.substring(0, 8);
					currentBuild.displayName = "#${env.BUILD_NUMBER} | ${env.NODE_NAME}"
						if (env.BRANCH_NAME ==~ /(master)/) {
							bat('%GIT_PYTHON_GIT_EXECUTABLE% log -1 --pretty=format:%%s>commit_log.txt')							
					//  	def message = readFile 'commit_log.txt'
                         // def reg_result = ("${message}" =~ /PR #[0-9]+/)[0]
                        }
                        else {
                            currentBuild.description = "BUILD: ${env.BRANCH_NAME} [${shortGitCommit}]"  // BUILD: 2020/devops/10393807_cmake_fix
                        }
                        FAILED_STAGE='checkBuildEnvironment'
                        print """
                        =========================================================================
                        CHECK Tool installation: Packager (prj_config/config_package.json)
                        =========================================================================
                        echo "Current workspace is ${env.WORKSPACE}"
                        """
						try {
							bat("${env.PY_EXE} ${env.PYTHONPATH}/${env.PACKAGER} -v -c ${env.PYTHONPATH}/config_package_bookshelf.json check")
                        } 
						catch (Exception e) {
                            bat("${env.PY_EXE} ${env.PYTHONPATH}/${env.PACKAGER} -v -c ${env.PYTHONPATH}/config_package_bookshelf.json install")
                        }
                        bat('mkdir %WORKSPACE%\\FINISHED_STAGES2')
  
                    }
			    }
            }            
			stage('DaveTheDoorman') {
				steps {
					echo 'Dave the Doorman'
						script {
							if (env.BRANCH_NAME ==~ /(master)/)
							{
								print "master branch"
							}
							else if (env.TAG_NAME ==~ /(L2H4060_.*)/) {
								echo "Tag Name is : "
								echo env.TAG_NAME						
							}
							else {
							isBranchCorrect = (env.BRANCH_NAME ==~ /[1-9][0-9]{3}\/(Autosar|BSW|ARXML|bugfix|feature|task|planning|integration|devops)\/([1-9][0-9]+)_.*/)
								if (isBranchCorrect == false) {
									echo 'branch name Incorrect'
									error "Failed in DaveTheDoorman as Branch name doesnt meet naming criteria, Exiting ..."
										step {
											sh 'echo "Fail!"; exit 1'
										}
								}
							}
						}
				}
			}  
			/******************************************************************************************************
				*    .______    __    __   __   __       _______
				*    |   _  \  |  |  |  | |  | |  |     |       \
				*    |  |_)  | |  |  |  | |  | |  |     |  .--.  |
				*    |   _  <  |  |  |  | |  | |  |     |  |  |  |
				*    |  |_)  | |  `--'  | |  | |  `----.|  '--'  |
				*    |______/   \______/  |__| |_______||_______/
				*
			******************************************************************************************************/			             
			stage('Build with Batchscript') {
				//Build stage can run the build application
				when {
					not {
						buildingTag()
					}
            	}
				steps{
					script {						
						if (params.qacCheckAndQAVDashboard == false && params.ReleaseNotes == false ) {	
							echo 'Build run'				  
							dir("${env.WORKSPACE}\\sw\\build\\") { 						                							
								bat (".\\Windows_build.bat")
							}
						}					
					}
				}
            }
			stage('QAC report generation and Upload to QAV DAshboard') {
				when {
					anyOf {
						buildingTag()
						expression { params.qacCheckAndQAVDashboard == true };						
                    }
				}				
				steps{
					script {												
						echo 'QAC build run'
							dir("${env.WORKSPACE}\\tools\\Qac_cpp_win/\\")  {  																
									bat (".\\GenReport.bat")								   
								if (env.TAG_NAME) {   									         								
									bat (".\\QavUpload.bat")
								} 
							}									                   																	
					}
            	}
			}
			stage("Release Notes") {
				when { expression { return params.ReleaseNotes } }
					steps {
						script {
							print """
								=========================================================================
									Release notes
								=========================================================================
							"""
							if (PreviousTagName.trim().isEmpty() || LatestTagName.trim().isEmpty()) {
								print "ERROR: PreviousTagName or LatestTagName parameters were not set"
								print "INFO: Please set both parameters for release notes"
								catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
									bat 'exit -300'
								}
							}
							else {
								dir("${env.WORKSPACE}\\config\\Jenkins\\py_scripts\\") {
									def buildResultVal = 'SUCCESS'

									//If running in master branch, fail the entire build
									if (env.BRANCH_NAME ==~ /(master)/) {
										buildResultVal = 'FAILURE'
									}
									withEnv(["PYTHONPATH=${env.WORKSPACE}\\config\\Jenkins\\py_scripts\\"]) {
										catchError(buildResult: buildResultVal, stageResult: 'FAILURE') {
											//Call setup.bat to install required python modules
											bat('setup.bat')
											if (ReleaseVersion.trim().isEmpty()) {
												print "WARNING: Release version is empty, using default value instead"
												bat(script: "${env.PY_EXE} buildprj.py --gitreldoc ${params.PreviousTagName} ${params.LatestTagName} TestRelease")
											}
											else {
												bat(script: "${env.PY_EXE} buildprj.py --gitreldoc ${params.PreviousTagName} ${params.LatestTagName} ${params.ReleaseVersion}")
											}
										}
									}
								}
							}
							archiveArtifacts artifacts: 'config/Jenkins/py_scripts/TestRelease.docm', onlyIfSuccessful: true
						}
						
					}
			}
			stage('Archive Build Artifacts') {
                steps {
                    script {
                        FAILED_STAGE='Archive Artifacts'
                        print """
                        =========================================================================
                        POST: Archive Artifacts
                        =========================================================================
                            """
                            archiveArtifacts allowEmptyArchive: true, artifacts: 'sw/binary/fisker_hydra_mcu1_0_boot_app_rtos/bin/j721s2_hyd/*'  
			    archiveArtifacts allowEmptyArchive: true, artifacts: 'sw/binary/fisker_hydra_app_mcu2_1/bin/j721s2_hyd/*' 
			    archiveArtifacts allowEmptyArchive: true, artifacts: 'tools/Qac_cpp_win/prqa/configs/Initial/reports/*.html'							                      
                    }
                }
            }
			stage ('Quality Evaluation'){
				parallel {
					/******************************************************************************************************
					*      ______           ___       ______
					*     /  __  \         /   \     /      |
					*    |  |  |  |       /  ^  \   |  ,----'
					*    |  |  |  |      /  /_\  \  |  |
					*    |  `--'  '--.  /  _____  \ |  `----.
					*     \_____\_____\/__/     \__\ \______|
					*
					******************************************************************************************************/       
					stage('QAC Test') {						                
						steps {
								script { 
									print """
									=========================================================================
										QAC Test
									=========================================================================
									"""									
								}
						}
					}
					/******************************************************************************************************
					*         _______. __   __             .___________. _______     _______.___________.
					*        /       ||  | |  |            |           ||   ____|   /       |           |
					*       |   (----`|  | |  |      ______`---|  |----`|  |__     |   (----`---|  |----`
					*        \   \    |  | |  |     |______|   |  |     |   __|     \   \       |  |
					*    .----)   |   |  | |  `----.           |  |     |  |____.----)   |      |  |
					*    |_______/    |__| |_______|           |__|     |_______|_______/       |__|
					*
					******************************************************************************************************/
					stage('SIL') {
						steps {
								echo 'SIL Test'				     
						}
					}	     
				}
			}
		}
		post {
			failure {           
				script {
					print("Post Jenkins Stage")	 
					def mail_recipient_integration="${EMAIL_ID}"
					env.mail_committer=mail_recipient_integration + env.COMMITTER_EMAIL
					env.mail_Integration=mail_recipient_integration
					print "Commiter Mail_ID"
					print ("${env.COMMITTER_EMAIL}")
					print ("${env.mail_committer}") 
					if (env.BRANCH_NAME != 'master' && buildingTag() != true) {      
						emailext ( 					
							mimeType: 'text/html',														
							subject: '$PROJECT_NAME - Build # $BUILD_NUMBER',
							body: """Hi Committer and integration team,
										Please check the failed build at:
										${env.BUILD_URL}
										""",                           
							to: "${env.mail_committer}"
						)
					}
					else {      
						emailext ( 					
							mimeType: 'text/html',														
							subject: '$PROJECT_NAME - Build # $BUILD_NUMBER',
							body: """Hi Committer and integration team,
										Please check the failed build at:
										${env.BUILD_URL}
										""",                           
							to: "${env.mail_Integration}"
						)
					}
				}
			}
		}
	}

