#!groovy
node('EPBYMINW2466') {

stage('Checking out') {
git branch: 'akarzhou', url: 'https://github.com/MNT-Lab/mntlab-pipeline.git'
	}	
stage ('Building code') {
sh "gradle build"
}

stage('Testing code') {
	parallel (
	Branch1: {
	stage ('Cucumber')
		{
			sh "/opt/gradle/bin/gradle cucumber"
		}
	}
	Branch2: {
        stage ('Unit Tests')
                {
                        sh "/opt/gradle/bin/gradle test"
                }
        }
	Branch3: {
        stage ('Jacoco Tests')
                {
                        sh "/opt/gradle/bin/gradle jacocoTestReport"
                }
	   }
	)
	}
//stage('Triggering job and fetching artefact after finishing') {
//	build job: 'MNTLAB-akarzhou-child1-build-job', parameters: [string(name: 'BRANCH_NAME', value: 'akarzhou')]
//	}
}

