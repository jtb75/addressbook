pipeline {
    agent { label 'dind-ssh-agent' }
    stages {
        stage('Build') {
            steps {
                echo 'Building Image..'
                sh """
                docker build -t webapps/addressbook:latest .
                docker tag webapps/addressbook:latest webapps/addressbook:$BUILD_NUMBER
                """
            }
        }
        stage('Scan') {
            steps {
                echo 'Scanning Image..'
                prismaCloudScanImage ca: '',
                cert: '',
                dockerAddress: 'tcp://192.168.1.215:2376',
                ignoreImageBuildTime: true,
                image: 'webapps/addressbook:$BUILD_NUMBER',
                key: '',
                logLevel: 'info',
                podmanPath: '',
                project: '',
                resultsFile: 'prisma-cloud-scan-results.json'
            }
        }
        stage ('Publish') {
            steps {
                echo 'Publishing Results..'
                prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
            }
        }
        stage ('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor_cred', passwordVariable: 'HARBOR_PW', usernameVariable: 'HARBOR_USER')]) {
                    echo 'Pushing Image to Registry..'
                    sh """
                    docker tag webapps/addressbook:latest 192.168.1.211:80/webapps/addressbook:$BUILD_NUMBER
                    docker tag webapps/addressbook:latest 192.168.1.211:80/webapps/addressbook:latest
                    docker login --username ${HARBOR_USER} --password ${HARBOR_PW} 192.168.1.211:80
                    docker push 192.168.1.211:80/webapps/addressbook:$BUILD_NUMBER
                    docker push 192.168.1.211:80/webapps/addressbook:latest
                    """
                }
            }
        }
        stage ('Cleanup') {
            steps {
                echo 'Cleaning up Image..'
                sh """
                docker rmi 192.168.1.211:80/webapps/addressbook:$BUILD_NUMBER
                docker rmi 192.168.1.211:80/webapps/addressbook:latest
                docker rmi webapps/addressbook:$BUILD_NUMBER
                docker rmi webapps/addressbook:latest
                """
            }
        }
    }
}
