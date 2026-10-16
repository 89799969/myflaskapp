pipeline {
    agent any

    environment {
        PYTHON_PATH = 'C:/Users/yanghaoyv/AppData/Local/Programs/Python/Python311/python.exe'
    }

    stages {
        stage('Verify Python Path') {
            steps {
                bat '''
                    @echo off
                    echo "=== 验证Python路径及版本 ==="
                    %PYTHON_PATH% --version
                    echo "Python路径验证通过！"
                '''
            }
        }
        stage('Checkout') {
            steps {
                retry(5) {
                    git url: 'git@github.com-jenkins:89799969/myflaskapp.git', branch: 'main'
                }
            }
        }
        stage('Fix pip') {
            steps {
                bat '''
                    @echo off
                    echo "=== 修复pip环境 ==="
                    %PYTHON_PATH% -m ensurepip --upgrade
                    %PYTHON_PATH% -m pip --version
                '''
            }
        }
        stage('Install Dependencies') {
            steps {
                bat '''
                    @echo off
                    %PYTHON_PATH% -m pip install --upgrade pip
                    %PYTHON_PATH% -m pip install -r requirements.txt
                '''
            }
        }
        stage('Lint') {
            steps {
                bat '%PYTHON_PATH% -m pip install flake8 && %PYTHON_PATH% -m flake8 app.py tests/'
            }
        }
        stage('Test') {
            steps {
                bat '''
                    %PYTHON_PATH% -m pip install pytest
                    %PYTHON_PATH% -m pytest --cov=app tests/ --cov-report=html
                '''
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'htmlcov',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        stage('Build') {
            steps {
                bat '''
                    %PYTHON_PATH% -m pip install pyinstaller
                    %PYTHON_PATH% -m PyInstaller --onefile app.py
                '''
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                bat '''
                    @echo off
                    for /f "tokens=5" %%a in ('netstat -aon ^| findstr ":5000" ^| findstr "LISTENING"') do taskkill /F /PID %%a
                    start "" "%PYTHON_PATH%" app.py
                '''
            }
        }
    }
    post {
        success { echo 'CI/CD pipeline completed successfully!' }
        failure { echo 'CI/CD pipeline failed!' }
    }
}
