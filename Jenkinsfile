pipeline {
  agent any
  stages {
    stage('Kill Running Processes') {
      steps {
        sh '''
        pkill -f dotnet || true
        '''
      }
    }

    stage('Clean Workspace') {
      steps {
        sh '''
        find . -type d -name "bin" -exec rm -rf {} +
        find . -type d -name "obj" -exec rm -rf {} +
        find . -type d -name "tmp-webcil" -exec rm -rf {} +
        '''
      }
    }

    stage('Clear NuGet Cache') {
      steps {
        sh 'dotnet nuget locals all --clear'
      }
    }

    stage('Restore') {
      steps {
        sh 'dotnet restore --ignore-failed-sources --disable-integrity-check'
      }
    }

    stage('Build') {
      steps {
        sh 'dotnet build --no-restore'
      }
    }

    stage('Tests') {
      parallel {
        stage('Unit') {
          steps {
            warnError(message: 'Unit problem') {
              sh 'dotnet test tests/UnitTests --no-build --logger "trx;LogFileName=unit-tests.trx"'
            }

          }
        }

        stage('Integration') {
          steps {
            sh 'dotnet test tests/IntegrationTests --no-build --logger "trx;LogFileName=integration-tests.trx"'
          }
        }

        stage('Functional') {
          steps {
            warnError(message: 'Functional problem') {
              sh 'dotnet test tests/FunctionalTests --no-build --logger "trx;LogFileName=functional-tests.trx"'
            }

          }
        }

      }
    }

    stage('Deployment') {
      steps {
        sh 'dotnet publish eShopOnWeb.sln -o /var/aspnet'
        dir(path: '/var/aspnet') {
          archiveArtifacts(artifacts: '*', onlyIfSuccessful: true)
        }

      }
    }

  }
  post {
    always {
      archiveArtifacts(artifacts: '**/*.trx', allowEmptyArchive: true)
    }

  }
}