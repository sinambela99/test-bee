pipeline {
    agent any
    environment {
        credential = 'id_rsa'
        server = 'baiksekali@103.150.92.227'
        directory = '/home/baiksekali/test-bee'
	directory2 = '/var/jenkins_home/workspace/test-be'
        branch = 'main'
        service = 'backend'
        image = 'iansinambela/be'
        SONARQUBE_URL = 'http://103.175.219.100:9000'
        SONARQUBE_TOKEN = '7904068fe98aad7c33a53f9fd0bdbef62f166046'
        SONARQUBE_PROJECT_KEY = 'ian'
    }
    stages {
        stage('Pull code dari repository') {
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    git pull origin ${branch}
                    exit
                    EOF'''
                }
            }
        }
	stage('SonarQube Analysis') {
	    environment {
	        SCANNER_HOME = tool 'sonarqube'
	    }
	    steps {
	        script {
	            withSonarQubeEnv('sonarqube') {
	                sh """
	                ${SCANNER_HOME}/bin/sonar-scanner \
	                -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
	                -Dsonar.sources=${directory} \
	                -Dsonar.host.url=${SONARQUBE_URL} \
	                -Dsonar.login=${SONARQUBE_TOKEN} \
			-Dsonar.ce.javaAdditionalOpts="--add-opens java.base/java.lang=ALL-UNNAMED" 
	                """
	            }
	        }
	    }
	}
        stage('Building application') {
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker compose down
                    docker compose up -d database
                    docker build -t ${image}:${BUILD_NUMBER} .
                    exit
                    EOF'''
                }
            }
        }
        stage('Testing application') {
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker stop test-bee || true
                    docker run --name test-bee -p 5000:5000 -d ${image}:${BUILD_NUMBER}
                    wget --spider localhost:5000
                    docker stop test-bee
                    docker rm test-bee
                    exit
                    EOF'''
                }
            }
        }
        stage('Deploy aplikasi on top docker') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    sed -i 's|image: .*$|image: ${image}:${BUILD_NUMBER}|' docker-compose.yaml
                    docker compose up -d ${service}
                    exit
                    EOF'''
                }
            }
        }
        stage('Push image to docker hub') {
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF
                    cd ${directory}
                    docker push ${image}:${BUILD_NUMBER}
                    docker rmi -f ${image}:${BUILD_NUMBER}
                    exit
                    EOF'''
                }
            }
        }
        stage('send notification to discord') {
            steps {
                discordSend description: "backend notify", footer: "ian notify", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: "https://discord.com/api/webhooks/1232551770614665298/xQdk4sfscxduagJVQ6gdpN1aYAXCIKr-D_L2fALi9pc0qUdcDNTMgq_vHzrxPxpOT-4V"
            }
        }
    }
    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if the pipeline succeeds'
        }
        failure {
            echo 'This will run only if the pipeline fails'
        }
    }
}

