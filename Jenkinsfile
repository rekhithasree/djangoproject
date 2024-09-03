node {
    try {
        stage('Checkout') {
            checkout scm
            sh 'git log HEAD^..HEAD --pretty="%h %an - %s" > GIT_CHANGES'
            def lastChanges = readFile('GIT_CHANGES')
            slackSend color: "warning", message: "Started `${env.JOB_NAME}#${env.BUILD_NUMBER}`\n\n_The changes:_\n${lastChanges}"
        }

        stage('Test') {
            withCredentials([string(credentialsId: '0f0b387c-3fd1-4329-abf4-d2a7bfe7b248', variable: 'DB_PASSWORD')]) {
                sh '''
                virtualenv env -p python3.10
                . env/bin/activate  
                pip install -r requirements.txt
                export DB_PASSWORD=${DB_PASSWORD}
                echo "DB_PASSWORD: ${DB_PASSWORD}"
                python manage.py test --testrunner=myproject.tests.test_runners.NoDbTestRunner
                '''
            }
        }

        stage('Deploy') {
            sh 'chmod +x ./deployment/deploy_prod.sh'
            sh './deployment/deploy_prod.sh'
        }

        stage('Publish results') {
            slackSend color: "good", message: "Build successful: `${env.JOB_NAME}#${env.BUILD_NUMBER}` <${env.BUILD_URL}|Open in Jenkins>"
        }
    } catch (err) {
        slackSend color: "danger", message: "Build failed :face_with_head_bandage: \n`${env.JOB_NAME}#${env.BUILD_NUMBER}` <${env.BUILD_URL}|Open in Jenkins>"
        throw err
    }
}
