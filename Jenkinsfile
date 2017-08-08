node(env.SLAVE) 
	{
	//env.gradle='/opt/gradle/gradle-4.1/bin/gradle'
    env.student='nzubkov'
    stage('Checkout code from repo') 
		{
			checkout([
				$class: 'GitSCM', 
				branches: [[name: '*/nzubkov']], 
				doGenerateSubmoduleConfigurations: false, 
				extensions: [], 
				submoduleCfg: [], 
				userRemoteConfigs: [[url: 'https://github.com/nikitozzz/mntlab-pipeline.git']]
				])
        }
    stage('Building code') 
		{	
			sh "/opt/gradle/gradle-4.1/bin/gradle -v"
			sh "gradle build"
		}
    stage('Testing code') 
		{
			parallel(Unittests:
				{
					stage ('Unit Tests') 
						{
							sh "gradle test"
						}		
				},
					jacocoTests: 
						{
							stage('Jacoco Tests') 
								{
									sh "gradle jacocoTestReport"
								}
						},
					cucumberTests: 
						{
							stage('Cucumber Tests') 
								{
									sh "gradle cucumber"
								}
						}
			)
		}
    stage('Triggering job and fetching artifact after finishing') 
		{
			build job: 'EPRURYAW0380-MNTLAB-zubkov-child1-build-job', parameters: [[$class: 'StringParameterValue', name: 'BRANCH_NAME', value: 'nzubkov']]
			step([$class: 'CopyArtifact', filter: 'nzubkov_dsl_script.tar.gz	', fingerprintArtifacts: true, flatten: true, projectName: 'EPRURYAW0380-MNTLAB-zubkov-child1-build-job', target: ''])
		}
    stage('Packaging and Publishing results') 
		{
			/*a)*/ sh 'tar xvf nzubkov_dsl_script.tar.gz'
			/*b)*/ sh 'tar zvfc pipeline-${student}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile -C build/libs/ gradle-simple.jar'
			/*c)*/ archiveArtifacts artifacts: 'pipeline-'+student+'-${BUILD_NUMBER}.tar.gz', allowEmptyArchive: false
			/*d)*/ sh 'curl -v -u admin:admin123 --upload-file pipeline-${student}-${BUILD_NUMBER}.tar.gz http://localhost/nexus/content/repositories/releases/'+student+'-${BUILD_NUMBER}.tar.gz'
		}
    stage('Asking for manual approval') 
		{
			timeout(time:5, unit:'DAYS') 
				{
					input 'Approve deployment?' //, submitter: 'student'
				}
		}
    stage('Deployment') 
		{
			sh 'java -jar build/libs/gradle-simple.jar'
        }
    }
    stage('Sending status') 
		{
			echo 'SUCCESS'
		}