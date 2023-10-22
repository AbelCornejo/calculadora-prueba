pipeline {
	agent any
	environment {

        DOCKERHUB_USER     = credentials('Abelcornejo')
        DOCKERHUB_PASSWORD = credentials('Abelcornejo1902.')

		PUPPET_MAIN_URL  = '35.184.65.50'
		PUPPET_AGENT_URL_DEV = "35.222.16.182"
		PUPPET_AGENT_URL_PROD = "34.122.242.179"

		PUPPET_MAIN_HOME = '/home/jenkins'
		PUPPET_AGENT_HOME = '/home/jenkins'

		PUPPET_MAIN_MANIFEST_DIR = '/etc/puppet/code/environments/production/manifests'
		PUPPET_MAIN_MODULE_MANIFEST_DIR = '/etc/puppet/code/environments/production/modules/mymodule/manifests'
		PUPPET_MAIN_DEV_FILES_DIR = '/etc/puppet/code/environments/production/modules/mymodule/files'
		PUPPET_MAIN_PROD_FILES_DIR = '/etc/puppet/code/environments/production/modules/mymodule/files'

    }
	stages {
		stage("BuildTests") {
			when {
				branch 'develop'
			}
			steps {
				echo 'Building tests...'
				sh '''
					docker-compose -f docker-compose-tests.yml down
					docker-compose -f docker-compose-tests.yml build
				'''
			}
		}
		stage("RunTests") {
			when {
				branch 'develop'
			}
			steps {
				echo 'Running tests...'
				sh '''
					docker-compose -f docker-compose-tests.yml up --exit-code-from SumaTest SumaTest
				'''
			}
		}
		stage("Build") {
			when {
				branch 'develop'
			}
			steps {
				echo 'Building docker images for deployment...'
				sh '''
					docker-compose -f docker-compose-dev.yml down
					docker-compose -f docker-compose-dev.yml build
				'''
			}
		}
		stage("PushBuilds") {
			when {
				branch 'develop'
			}
			steps {
				echo "Pushing docker images to DockerHub..."
				sh '''
					docker login -u $DOCKERHUB_USER -p $DOCKERHUB_PASSWORD
					docker-compose -f docker-compose-dev.yml push
				'''
			}
		}
		stage("DeployDev") {
			when {
				branch 'develop'
			}
			steps {
				echo "Deploying to development..."
				sh '''

					echo "New deployment" >> deployments.txt
					scp deployments.txt jenkins@${PUPPET_AGENT_URL_DEV}:${PUPPET_AGENT_HOME}/
					
					scp docker-compose-dev.yml jenkins@${PUPPET_MAIN_URL}:${PUPPET_MAIN_HOME}/docker-compose.yml
					scp site.pp jenkins@${PUPPET_MAIN_URL}:${PUPPET_MAIN_HOME}/
					scp init.pp jenkins@${PUPPET_MAIN_URL}:${PUPPET_MAIN_HOME}/

					ssh jenkins@${PUPPET_MAIN_URL} sudo mv ${PUPPET_MAIN_HOME}/docker-compose.yml ${PUPPET_MAIN_DEV_FILES_DIR}/
					ssh jenkins@${PUPPET_MAIN_URL} sudo mv ${PUPPET_MAIN_HOME}/site.pp ${PUPPET_MAIN_MANIFEST_DIR}/
					ssh jenkins@${PUPPET_MAIN_URL} sudo mv ${PUPPET_MAIN_HOME}/init.pp ${PUPPET_MAIN_MODULE_MANIFEST_DIR}/


				'''
			}
		}
		stage("DeployProd") {
			when {
				branch 'main'
			}
			steps {
				echo 'Deploying to production...'
				sh '''

					echo "New deployment" >> deployments.txt
					scp deployments.txt jenkins@${PUPPET_AGENT_URL_PROD}:${PUPPET_AGENT_HOME}/

					scp docker-compose-prod.yml jenkins@${PUPPET_MAIN_URL}:${PUPPET_MAIN_HOME}/docker-compose.yml
					scp site.pp jenkins@${PUPPET_MAIN_URL}:${PUPPET_MAIN_HOME}/
					scp init.pp jenkins@${PUPPET_MAIN_URL}:${PUPPET_MAIN_HOME}/

					ssh jenkins@${PUPPET_MAIN_URL} sudo mv ${PUPPET_MAIN_HOME}/docker-compose.yml ${PUPPET_MAIN_DEV_FILES_DIR}/
					ssh jenkins@${PUPPET_MAIN_URL} sudo mv ${PUPPET_MAIN_HOME}/site.pp ${PUPPET_MAIN_MANIFEST_DIR}/
					ssh jenkins@${PUPPET_MAIN_URL} sudo mv ${PUPPET_MAIN_HOME}/init.pp ${PUPPET_MAIN_MODULE_MANIFEST_DIR}/
					
				''' 
			}
		}
	}
}