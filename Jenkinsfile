pipeline {
  agent any

  stages {
    stage('Kill Running Processes') {
      steps {
        // Tuer tous les processus dotnet en cours
        sh '''
        pkill -f dotnet || true
        '''
      }
    }

    stage('Clean Workspace') {
      steps {
        // Nettoyer les fichiers temporaires pour éviter les conflits
        sh '''
        find . -type d -name "bin" -exec rm -rf {} +
        find . -type d -name "obj" -exec rm -rf {} +
        find . -type d -name "tmp-webcil" -exec rm -rf {} +
        '''
      }
    }

    stage('Clear NuGet Cache') {
      steps {
        // Vider le cache NuGet
        sh 'dotnet nuget locals all --clear'
      }
    }

    stage('Restore') {
      steps {
        // Restaurer les packages NuGet en ignorant les avertissements
        sh 'dotnet restore --ignore-failed-sources'
      }
    }

    stage('Build') {
      steps {
        // Compiler le projet sans restaurer les packages (déjà fait)
        sh 'dotnet build --no-restore'
      }
    }

    stage('Tests') {
      parallel {
        stage('Unit') {
          steps {
            // Exécuter les tests unitaires
            sh 'dotnet test tests/UnitTests --no-build --logger "trx;LogFileName=unit-tests.trx"'
          }
        }

        stage('Integration') {
          steps {
            // Exécuter les tests d'intégration
            sh 'dotnet test tests/IntegrationTests --no-build --logger "trx;LogFileName=integration-tests.trx"'
          }
        }

        stage('Functional') {
          steps {
            // Exécuter les tests fonctionnels
            sh 'dotnet test tests/FunctionalTests --no-build --logger "trx;LogFileName=functional-tests.trx"'
          }
        }
      }
    }

    stage('Deployment') {
      steps {
        // Publier l'application dans le dossier spécifié
        sh 'dotnet publish eShopOnWeb.sln -o /var/aspnet'
      }
    }
  }

  post {
    always {
      // Archiver les résultats des tests pour une analyse ultérieure
      archiveArtifacts artifacts: '**/*.trx', allowEmptyArchive: true
    }
  }
}
