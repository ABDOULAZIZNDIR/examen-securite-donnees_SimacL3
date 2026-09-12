pipeline {
    agent any
    tools {
        nodejs 'NodeJS 20'
    }
    stages {
        stage('Clone OWASP Juice Shop') {
            steps {
                git branch: 'master', url: 'https://github.com/ABDOULAZIZNDIR/juice-shop.git'
            }
        }
        stage('Installer les dépendances') {
            steps {
                sh 'npm install'
            }
        }
        stage('Scan de sécurité - npm audit') {
            steps {
                sh 'npm audit --json > npm-audit-report.json || true'
            }
        }
        stage('Scan OWASP Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '--scan . --format HTML --format XML --nvdApiKey 67905f75-efbd-4936-b401-02b3f03f8dca', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
        stage('Secret Detection - Gitleaks') {
            steps {
                sh 'docker run --rm -v $(pwd):/repo zricethezav/gitleaks:latest detect --source="/repo" --report-path="/repo/gitleaks-report.json" --no-git || true'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/dependency-check-report.*, npm-audit-report.json, gitleaks-report.json', allowEmptyArchive: true
            publishHTML(target: [
                reportDir: '.',
                reportFiles: 'dependency-check-report.html',
                reportName: 'Rapport de sécurité OWASP'
            ])
            emailext (
                subject: "Rapport de sécurité Juice Shop - Build #${env.BUILD_NUMBER}",
                body: "Le scan de sécurité est terminé. Consultez le rapport joint ou dans Jenkins.",
                to: 'azid.x69@gmail.com',
                attachmentsPattern: 'dependency-check-report.html, npm-audit-report.json, gitleaks-report.json'
            )
        }
    }
}
