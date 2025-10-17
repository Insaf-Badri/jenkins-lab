pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')
        VENV_DIR = ".venv"
        HOST = "0.0.0.0"
        PORT = "5000"
        APP_MODULE = "app:app"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/hamiiiid02/jenkins-lab.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                bat """
                python -m venv %VENV_DIR%
                call %VENV_DIR%\\Scripts\\activate
                python -m pip install --upgrade pip
                pip install -r requirements.txt
                """
            }
        }

        stage('Check Python') {
            steps {
                bat 'python --version'
            }
        }

        stage('Run Tests') {
            steps {
                bat """
                call %VENV_DIR%\\Scripts\\activate
                python -m pytest test_app.py -v
                """
            }
        }

        stage('Build') {
            steps {
                bat 'echo "Build step placeholder (Flask apps usually don\'t need build)"'
            }
        }

        stage('Deploy (Local with Gunicorn)') {
            steps {
                echo 'Starting Flask app locally using Gunicorn...'
                bat """
                call %VENV_DIR%\\Scripts\\activate
                start /B gunicorn --bind %HOST%:%PORT% %APP_MODULE% > gunicorn.log 2>&1
                echo Gunicorn started on http://%HOST%:%PORT%
                """
            }
        }

        stage('Test SMTP Connection') {
            steps {
                bat """
                echo Testing SMTP...
                powershell -Command "Test-NetConnection smtp.gmail.com -Port 465"
                """
            }
        }
    }

    post {
        success {
            emailext(
                to: "hamdonhamid67@gmail.com",
                from: "amineallali9@gmail.com",
                replyTo: "amineallali9@gmail.com",
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """\
                <p>Good news!</p>
                <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> succeeded.</p>
                <p>Check details: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }

        failure {
            emailext(
                to: "hamdonhamid67@gmail.com",
                from: "amineallali9@gmail.com",
                replyTo: "amineallali9@gmail.com",
                subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """\
                <p>Uh oh...</p>
                <p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> failed.</p>
                <p>Check logs: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """
            )
        }
    }
}
