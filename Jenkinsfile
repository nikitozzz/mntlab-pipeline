node(env.SLAVE) 
	{
	//env.gradle='/opt/gradle/gradle-4.1/bin/gradle'
    env.student='nzubkov'
	//env.datestamp='$(date +%F)'
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
			sh "/opt/gradle/gradle-4.1/bin/gradle build"
		}
    stage('Testing code') 
		{
			parallel(Unittests:
				{
					stage ('Unit Tests') 
						{
							sh "/opt/gradle/gradle-4.1/bin/gradle test"
						}		
				},
					jacocoTests: 
						{
							stage('Jacoco Tests') 
								{
									sh "/opt/gradle/gradle-4.1/bin/gradle jacocoTestReport"
								}
						},
					cucumberTests: 
						{
							stage('Cucumber Tests') 
								{
									sh "/opt/gradle/gradle-4.1/bin/gradle cucumber"
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
			sh 'tar xvf nzubkov_dsl_script.tar.gz'
			sh 'tar zvfc pipeline-${student}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile -C build/libs/ gradle-simple.jar'
			archiveArtifacts artifacts: 'pipeline-'+student+'-${BUILD_NUMBER}.tar.gz', allowEmptyArchive: false
		}
    stage('Asking for manual approval') 
		{
			timeout(time:10, unit:'SECONDS')
				{
					input 'Approve deployment artefact?'
				}
		}
	stage ('Deployment artefact')
		{
			sh 'curl -v -u admin:admin123 --upload-file pipeline-${student}-${BUILD_NUMBER}.tar.gz http://localhost:8081/nexus/content/repositories/releases/'+student+'-${BUILD_NUMBER}.tar.gz'
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