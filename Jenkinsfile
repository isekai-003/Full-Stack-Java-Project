pipeline {
    
	agent {
        label 'node1'}
    
	
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
		NEXUSIP = '13.127.110.198'
		NEXUSPORT = '8081'
		NEXUS_GRP_REPO = 'vpro-mvn-group'
        NEXUS_LOGIN = 'nexusip'
        ARTVERSION = "${env.BUILD_ID}"
        NEXUSPASS = credentials('nexuspass')
        
    }
	
    stages{
        
        stage('BUILD'){
            agent { label 'node1' }
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

	stage('UNIT TEST'){
        agent { label 'node1' }
            steps {
                sh 'mvn test'
            }
        }
    
	// stage('INTEGRATION TEST'){
    //     agent { label 'node1' }
    //         steps {
    //             sh 'mvn verify -DskipUnitTests'
    //         }
    //     }
   
        
    //     stage ('CODE ANALYSIS WITH CHECKSTYLE'){
    //         agent { label 'node1' }
    //         steps {
    //             sh 'mvn checkstyle:checkstyle'
    //         }
    //         post {
    //             success {
    //                 echo 'Generated Analysis Result'
    //             }
    //         }
    //     }
    // //     stage('OWASP Dependency Check') {
    // //         agent { label 'node1' }
    // //         steps {
    // //              dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DP'
    // //              dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    // //         }
    // // }

    //     stage('CODE ANALYSIS with SONARQUBE') {
    //         agent { label 'node1' }
          
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
            agent { label 'node2' }
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
    
    
    
      stage('Build and Push Docker Image') {
        agent {  label 'node1' }

      environment {
        DOCKER_IMAGE = "shamshuddin03/vpro:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('dockerlogin')
      }
      steps {
        script {
            sh ' docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "dockerlogin") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Trivy') {
        agent {  label 'node1' }
            steps {
                 sh "trivy fs ."
            }
        }
        
         stage('Update Deployment File') {
            agent {  label 'node1' }
        environment {
            GIT_REPO_NAME = "Spring-Boot-Kubernetes"
            GIT_USER_NAME = "isekai-003"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
             checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/isekai-003/Spring-Boot-Kubernetes.git']])

                sh '''
                    git config user.email "shamshuddin0003@gmail.com"
                    git config user.name "isekai-003"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" helm/vprofilecharts/values.yaml
                    git add .
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
    

           
    }
    }
       


    



