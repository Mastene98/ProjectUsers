//azerty

pipeline {
    agent any
    
    parameters {
        choice(
            name: 'ENVIRONNEMENT',
            choices: ['Test', 'PP', 'Prod'],
            description: 'Choisir l’environnement Postman à utiliser'
        )
    }

    environment {
        RAPPORT_HTML = 'reports/RapportCollectionClients.html'
        NEWMAN_EXIT_CODE = '0'
    }

    stages {

        stage('1 - Recuperation du projet GitHub') {
            steps {
                echo 'Recuperation du depot GitHub...'
                git branch: 'main', url: 'https://github.com/Mastene98/ProjectUsers.git'
            }
        }

        stage('2 - Verification de Node.js') {
            steps {
                echo 'Verification de Node.js...'
                bat 'node -v'
            }
        }

        stage('3 - Verification de npm') {
            steps {
                echo 'Verification de npm...'
                bat 'npm -v'
            }
        }

        stage('4 - Verification / Installation de Newman') {
            steps {
                echo 'Verification de Newman...'

                bat '''
                where newman >nul 2>&1
                if %ERRORLEVEL% NEQ 0 (
                    echo Newman non trouve. Installation de Newman...
                    npm install newman --no-save
                ) else (
                    echo Newman est deja installe.
                )
                '''

                echo 'Version de Newman utilisee :'
                bat 'npx newman -v'
            }
        }

        stage('5 - Verification / Installation du reporter HTML') {
            steps {
                echo 'Verification du reporter newman-reporter-htmlextra...'

                bat '''
                if not exist node_modules\\newman-reporter-htmlextra (
                    echo Reporter HTML non trouve. Installation du reporter htmlextra...
                    npm install newman-reporter-htmlextra --no-save
                ) else (
                    echo Reporter HTML deja present.
                )
                '''
            }
        }

        stage('6 - Preparation du dossier de rapports') {
            steps {
                echo 'Preparation du dossier reports...'
                bat 'if not exist reports mkdir reports'
            }
        }

        stage('7 - Execution de la collection Postman et generation du rapport') {
            steps {
                echo 'Execution de la collection Postman avec Newman...'

                script {
                    def status = bat(
                        script: "npx newman run Clients.postman_collection.json -e ${params.ENVIRONNEMENT}.postman_environment.json --reporters=cli,htmlextra --reporter-htmlextra-export ${env.RAPPORT_HTML}",
                        returnStatus: true
                    )

                    env.NEWMAN_EXIT_CODE = status.toString()

                    if (status != 0) {
                        echo "Newman a retourne le code ${status}"
                        currentBuild.result = 'UNSTABLE'
                    } else {
                        echo 'Tous les tests Postman sont passes.'
                    }
                }
            }
        }

        stage('8 - Verification du rapport HTML') {
            steps {
                echo 'Verification du rapport HTML...'

                bat '''
                if exist reports\\RapportCollectionClients.html (
                    echo Rapport HTML genere avec succes
                ) else (
                    echo Rapport HTML introuvable
                    exit /b 1
                )
                '''
            }
        }

        stage('9 - Archivage du rapport dans Jenkins') {
            steps {
                echo 'Archivage du rapport HTML...'
                archiveArtifacts artifacts: "${env.RAPPORT_HTML}", fingerprint: true
            }
        }

        stage('10 - Gestion du statut final') {
            steps {
                script {
                    if (env.NEWMAN_EXIT_CODE != '0') {
                        echo 'Certains tests ont echoue.'
                        currentBuild.result = 'UNSTABLE'
                    } else {
                        echo 'Pipeline terminee avec succes.'
                    }
                }
            }
        }

        stage('11 - Conservation des traces d execution') {
            steps {
                echo "Nom du job : ${env.JOB_NAME}"
                echo "Numero du build : ${env.BUILD_NUMBER}"
                echo "Resultat du build : ${currentBuild.currentResult}"
                echo "Lien du build : ${env.BUILD_URL}"
                echo "Rapport HTML : ${env.RAPPORT_HTML}"

                bat '''
                echo Date execution : %DATE%
                echo Heure execution : %TIME%
                '''
            }
        }

        stage('12 - Envoi du rapport par mail') {
            steps {
                echo 'Envoi du mail avec le rapport HTML...'

                emailext(
                    to: 'messouyamastene@gmail.com',
                    subject: "[Jenkins] ${currentBuild.currentResult} - ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                    body: """
Bonjour,

La pipeline Jenkins est terminee.

Projet : ${env.JOB_NAME}
Build : #${env.BUILD_NUMBER}
Statut : ${currentBuild.currentResult}

Lien :
${env.BUILD_URL}

Le rapport Newman HTML est joint a ce mail.

Cordialement,
Mastene Messouya Jenkins
""",
                    attachmentsPattern: "${env.RAPPORT_HTML}",
                    mimeType: 'text/plain'
                )
            }
        }
    }

    post {
        success {
            echo 'SUCCESS : pipeline terminee.'
        }

        unstable {
            echo 'UNSTABLE : certains tests ont echoue.'
        }

        failure {
            echo 'FAILURE : erreur technique.'
        }

        always {
            echo 'Fin de la pipeline.'
            echo "Lien du build : ${env.BUILD_URL}"
        }
    }
}