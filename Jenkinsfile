pipeline {
    agent any

    environment {
        // No changes needed here, Jenkins handles variables cross-platform
        GITHUB_TOKEN = credentials('github-token')
        VENV_DIR = ".venv" // The variable name is fine
        HOST = "0.0.0.0"
        PORT = "5000"
        APP_MODULE = "app:app"  
    }

    stages {
        stage('Checkout') {
            steps {
                // The git step is cross-platform
                git branch: 'main',
                    url: 'https://github.com/hamiiiid02/jenkins-lab.git',
                    credentialsId: 'github-token'
            }
        }

        stage('Setup Virtual Environment') {
            steps {
                // Use 'bat' for Windows batch scripts
                bat """
                echo "Creating Python virtual environment..."
                python -m venv %VENV_DIR%
                
                echo "Activating virtual environment and installing dependencies..."
                call %VENV_DIR%\\Scripts\\activate.bat
                pip install --upgrade pip
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
                call %VENV_DIR%\\Scripts\\activate.bat
                python -m pytest test_app.py -v
                """
            }
        }

        stage('Build') {
            steps {
                // 'echo' works in both batch and shell
                bat 'echo "Build step placeholder (Flask apps usually don\'t need build)"'
            }
        }

        stage('Deploy (Local with Waitress)') {
            steps {
                // Gunicorn doesn't run on Windows. Using Waitress instead.
                bat """
                echo "Installing waitress for Windows deployment..."
                call %VENV_DIR%\\Scripts\\pip.exe install waitress

                echo "Starting Flask app with Waitress..."
                start /B waitress-serve --host=%HOST% --port=%PORT% %APP_MODULE% > waitress.log 2>&1
                
                echo "✅ Waitress started on http://%HOST%:%PORT%"
                """
            }
        }
    
        stage('Test SMTP Connection') {
            steps {
                // 'nc' doesn't exist on Windows. Using PowerShell's Test-NetConnection.
                powershell '''
                if (Test-NetConnection -ComputerName smtp.gmail.com -Port 465 -WarningAction SilentlyContinue | Select-Object -ExpandProperty TcpTestSucceeded) {
                    Write-Host "✅ Connection to smtp.gmail.com on port 465 was successful."
                } else {
                    Write-Host "❌ Failed to connect to smtp.gmail.com on port 465."
                    exit 1
                }
                '''
            }
        }
    }

    post {
        // The emailext plugin step is cross-platform. No changes needed.
        success {
            emailext(
                to: "hamdonhamid67@gmail.com",
                from: "amineallali9@gmail.com",
                replyTo: "amineallali9@gmail.com",
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Good news!</p><p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> succeeded.</p><p>Check details: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>"""
            )
        }
    
        failure {
            emailext(
                to: "hamdonhamid67@gmail.com",
                from: "amineallali9@gmail.com",
                replyTo: "amineallali9@gmail.com",
                subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Uh oh...</p><p>Build <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> failed.</p><p>Check logs: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>"""
            )
        }
    }
}
