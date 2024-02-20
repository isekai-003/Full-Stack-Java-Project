pipeline {
    
	agent any
	
	tools {
        maven "maven3"
	jdk "java"	
    }
	
    environment {
      SNAP_REPO = 'vprofile-snapshot'
		NEXUS_USER = 'admin'
		NEXUS_PASS = 'akshay'
		RELEASE_REPO = 'vprofile-release'
		CENTRAL_REPO = 'vpro-mvn-central'
		NEXUSIP = '3.110.197.155'
		NEXUSPORT = '8081'
		NEXUS_GRP_REPO = 'vpro-mvn-group'
        NEXUS_LOGIN = 'nexusip'
        ARTVERSION = "${env.BUILD_ID}"
        NEXUSPASS = credentials('nexuspass')
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	// stage('UNIT TEST'){
    //         steps {
    //             sh 'mvn test'
    //         }
    //     }
    // stage('OWASP Dependency Check') {
    //         steps {
    //              dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DP'
    //              dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    //         }
    // }

	// stage('INTEGRATION TEST'){
    //         steps {
    //             sh 'mvn verify -DskipUnitTests'
    //         }
    //     }
   
        
    //     stage ('CODE ANALYSIS WITH CHECKSTYLE'){
    //         steps {
    //             sh 'mvn checkstyle:checkstyle'
    //         }
    //         post {
    //             success {
    //                 echo 'Generated Analysis Result'
    //             }
    //         }
    //     }

    //     stage('CODE ANALYSIS with SONARQUBE') {
          
	// 	  environment {
    //          scannerHome = tool 'sonar-scanner'
    //       }

    //       steps {
    //         withSonarQubeEnv('sonar-server') {
    //            sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
    //                -Dsonar.projectName=vprofile \
    //                -Dsonar.projectVersion=1.0 \
    //                -Dsonar.sources=src/ \
    //                -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
    //                -Dsonar.junit.reportsPath=target/surefire-reports/ \
    //                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
    //                -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
    //         }

    //         timeout(time: 10, unit: 'MINUTES') {
    //            waitForQualityGate abortPipeline: true
    //         }
    //       }
    //     }
           stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${RELEASE_REPO}",
                  credentialsId: "${NEXUS_LOGIN}",
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }
           stage('Ansible Deploy to staging') {
            steps {
                
             ansiblePlaybook ([
                credentialsId: 'applogin',
                inventory: 'Full-Stack-Java-Project/ansible/stage.inventory',
                playbook: 'Full-Stack-Java-Project/ansible/site.yml',
                 vaultTmpPath: ''
                    extraVars: [
                        USER: "admin",
                        PASS: "${NEXUSPASS}",
                        nexusip: "3.110.197.155",
                        reponame: "vprofile-release",
                        groupid: "QA",
                        time: "${env.BUILD_TIMESTAMP}",
                        build: "${env.BUILD_ID}",
                        artifactid: "vproapp",
                        vprofile_version: "vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"
                    ]
             ])
                  
                
            }
        }
    }
    }
       


    



