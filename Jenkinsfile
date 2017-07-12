#!/usr/bin/env groovy
 

node (env.SLAVE) {

	env.PATH=env.PATH+":/opt/gradle-3.5/bin:/opt/groovy/bin"
	env.ARTIFACT_SUFFIX="pipeline-aaksionkin"
	echo env.ARTIFACT_SUFFIX
	//env.ACTION="push"

stage 'Preparation (Checking out)'
	
	git url: "https://github.com/MNT-Lab/mntlab-pipeline.git", branch: 'aaksionkin'
	echo "1 Repo downloaded"
 

 
stage 'Building'
	
	sh 'gradle build'
	wrap([$class: 'TimestamperBuildWrapper']) {
     	echo "2 Build completed"
   }

stage 'Testing'

	parallel (
		'Cucumber Tests': {sh 'gradle cucumber' },
		'Jacoco Tests': {sh 'gradle jacocoTestReport' },
        'Unit Tests': {sh 'gradle test' }
		)
	wrap([$class: 'TimestamperBuildWrapper']) {
     	echo "3 Testing completed"
   }

stage ' Triggering job and fetching artifacts'

	build job: 'MNTLAB-aaksionkin-child1-build-job', parameters: [string(name: 'BRANCH_NAME', value: 'aaksionkin')], wait: true
	step([$class: 'CopyArtifact',
	  projectName: "EPBYMINW3088/MNTLAB-aaksionkin-child1-build-job",
	  filter: '*.tar.gz']);

	wrap([$class: 'TimestamperBuildWrapper']) {
     	echo "4 Job was triggered and finished"
   }

stage 'Packing and Publishing'

	sh 'tar -xzf aaksionkin_dsl_script.tar.gz'
	sh 'cp build/libs/mntlab-ci-pipeline.jar gradle-simple.jar'
	sh 'tar -zcf ${ARTIFACT_SUFFIX}-${BUILD_NUMBER}.tar.gz jobs.groovy Jenkinsfile gradle-simple.jar'
	sh 'groovy NexusPushPull'

	wrap([$class: 'TimestamperBuildWrapper']) {
     	echo "New artifact was published"
   }

} // node

