pipeline {
    agent any
    environment{
        credential = 'id_rsa'
        server = 'baiksekali@103.150.92.227'
        directory = '/home/baiksekali/test-bee'
        branch = 'main'
        service = 'backend'
        image = 'iansinambela/be'
    }
    stages {
        stage('Pull code dari repository'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    docker compose down
                    cd ${directory}
                    git pull origin ${branch}
                    exit
                    EOF'''
                }
            }
        }
        stage("SonarQube Analysis") {
            environment {
                SCANNER_HOME = tool 'sonarqube'  // Pastikan nama alat di konfigurasi alat Jenkins adalah 'sonarqube'
            }
            steps {
                script {
                    withSonarQubeEnv('ian') {  // Pastikan nama instalasi di konfigurasi sistem Jenkins adalah 'ian'
                        sh '''${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=ian \
                        -Dsonar.token=775a041b80fb88e526338c32b2466d76269269b4 \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://103.175.219.100:9000/ \
                        -Dsonar.inclusions=index.js \
                        -Dsonar.test.inclusions=index.test.js'''
                    }
                }
            }
        }
        stage('Building application'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
                    docker compose  up -d database
                    docker build -t ${image}:${BUILD_NUMBER} .
                    exit 
                    EOF'''
                }
            }
        }
        stage('Testing application'){
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF 
                    cd ${directory}
		    docker stop test-bee
                    docker run --name test-bee -p 5000:5000 -d ${image}:${BUILD_NUMBER}
		    wget --spider localhost:5000
		    docker stop test-bee
		    docker rm test-bee
		    exit
                    EOF'''
                }
            }
        }
        stage('Deploy aplikasi on top docker'){
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sshagent([credential]) {
                    sh '''ssh -o StrictHostKeyChecking=no ${server} << EOF
                    sed -i '20c\\    image: ${image}:${BUILD_NUMBER}' docker-compose.yaml
		    docker compose up -d backend 
                    cd ${directory}
                    exit
                    EOF'''
                }
            }
        }
        stage('Push image to docker hub'){
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
        stage('send notification to discord'){
            steps {
                discordSend description: "backend notify", footer: "ian notify", link: env.BUILD_URL,result: currentBuild.currentResult, title: JOB_NAME, webhookURL: "https://discord.com/api/webhooks/1232551770614665298/xQdk4sfscxduagJVQ6gdpN1aYAXCIKr-D_L2fALi9pc0qUdcDNTMgq_vHzrxPxpOT-4V"
            }
        }
    }
}
