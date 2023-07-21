pipeline {
    agent any
    tools {
        maven "MAVEN_HOME"
    }
	environment{
         IMAGE = readMavenPom().getArtifactId()     
         }
    stages {
        stage('Build') {
		
            environment {
                MULE_KEY = credentials('mule.key')
                ANYPOINT_CREDENTIALS = credentials('anypoint_connectedapp')
                NEXUS_CRDS = credentials('nexus_crds')
                BASIC_AUTH = credentials('basicAuth_cred')
            }
            steps {
                sh 'mvn -s settings.xml clean package -DUSERNAME=${ANYPOINT_CREDENTIALS_USR} -DPASSWORD=${ANYPOINT_CREDENTIALS_PSW} -Dnexus_user=${NEXUS_CRDS_USR} -Dnexus_pass=${NEXUS_CRDS_PSW} -Dkey=${MULE_KEY_PSW} -Denv=dev  -Dhttp.port=8081 -DRUNTIME_VERSION=4.4.0 -Duser=${BASIC_AUTH_USR} -Dpass=${BASIC_AUTH_PSW} -DTARGET=oxford-nonprod-privatespace'
            }
        }
        stage ('Deploy to Dev') {
		    environment {
                MULE_KEY = credentials('mule.key')
                ANYPOINT_CREDENTIALS = credentials('anypoint_connectedapp')
                NEXUS_CRDS = credentials('nexus_crds')  
                BASIC_AUTH = credentials('basicAuth_cred')            
            }
            when {
                expression { GIT_BRANCH ==~ /(origin\/dev)/ }
            }
            steps {
                sh 'mvn -P cloudhub -s settings.xml -DskipMunitTests  deploy -DUSERNAME=${ANYPOINT_CREDENTIALS_USR} -DPASSWORD=${ANYPOINT_CREDENTIALS_PSW} -DENVIRONMENT=Development -DAPPNAME=${IMAGE}-dev -DREPLICAS=1 -DVCORES=0.1 -Dkey=${MULE_KEY_PSW} -Denv=dev  -Dhttp.port=8081 -DRUNTIME_VERSION=4.4.0 -Duser=${BASIC_AUTH_USR} -Dpass=${BASIC_AUTH_PSW} -DTARGET=oxford-nonprod-privatespace'
				
                sh 'mvn -P cloudhub -s settings.xml -DskipMunitTests deploy -DmuleDeploy -DUSERNAME=${ANYPOINT_CREDENTIALS_USR} -DPASSWORD=${ANYPOINT_CREDENTIALS_PSW} -DENVIRONMENT=Development -DAPPNAME=${IMAGE}-dev  -DREPLICAS=1 -DVCORES=0.1 -Dkey=${MULE_KEY_PSW} -Denv=dev  -Dhttp.port=8081 -DRUNTIME_VERSION=4.4.0 -Duser=${BASIC_AUTH_USR} -Dpass=${BASIC_AUTH_PSW} -DTARGET=oxford-nonprod-privatespace'
            }
        }
        stage ('Deploy to Test') {
		    environment {
                MULE_KEY = credentials('mule.UATkey')
                ANYPOINT_CREDENTIALS = credentials('anypoint_connectedapp')
                NEXUS_CRDS = credentials('nexus_crds')
                BASIC_AUTH = credentials('basicAuth_UAT_cred')            
            }
            when {
                expression { GIT_BRANCH ==~ /(origin\/test)/ }
            }
            steps {            
                sh 'mvn -P cloudhub -s settings.xml -DskipMunitTests deploy -DmuleDeploy -DUSERNAME=${ANYPOINT_CREDENTIALS_USR} -DPASSWORD=${ANYPOINT_CREDENTIALS_PSW} -DENVIRONMENT=Test -DAPPNAME=${IMAGE}-test  -DREPLICAS=1 -DVCORES=0.1 -Dkey=${MULE_KEY_PSW} -Denv=test  -Dhttp.port=8081 -DRUNTIME_VERSION=4.4.0 -Duser=${BASIC_AUTH_USR} -Dpass=${BASIC_AUTH_PSW} -DTARGET=oxford-nonprod-privatespace'
            }
        }
        stage ('Deploy to Prod') {
		    environment {
                MULE_KEY = credentials('mule.key')
                ANYPOINT_CREDENTIALS = credentials('anypoint_connectedapp')
                NEXUS_CRDS = credentials('nexus_crds')
                BASIC_AUTH = credentials('basicAuth_cred')              
            }
            when {
                expression { GIT_BRANCH ==~ /(origin\/main)/ }
            }
            steps {				
                
            	emailext to: env.APPROVAL_EMAIL, subject: 'New build is waiting for your decision', body: 'Please make your decision about new build in Jenkins! \n\n Click on below link to redirect to approval page:- \n ${BUILD_URL}input/', attachLog: true
               script {
                        timeout(time: 600, unit: 'SECONDS') {
                    script {
                        env.RELEASE_TO_PROD = input message: 'User input required',
                            parameters: [choice(name: 'Promote to production', choices: 'No\nYes', description: 'Choose "Yes" if you want to deploy this build in prduction')]
                        milestone 1
                    }
                }
                        if (env.RELEASE_TO_PROD == "Yes") {
                            sh 'mvn -P cloudhub -s settings.xml -DskipMunitTests deploy -DmuleDeploy -DUSERNAME=${ANYPOINT_CREDENTIALS_USR} -DPASSWORD=${ANYPOINT_CREDENTIALS_PSW} -DENVIRONMENT=Production -DAPPNAME=${IMAGE}  -DREPLICAS=1 -DVCORES=0.1 -Dkey=${MULE_KEY_PSW} -Denv=prod  -Dhttp.port=8081 -DRUNTIME_VERSION=4.4.0 -Duser=${BASIC_AUTH_USR} -Dpass=${BASIC_AUTH_PSW} -DTARGET=oxford-prod-privatespace'
                        } else {
                            echo 'Production Deployment Rejected'
                            }
                }
            }
        }
    }
}