pipeline {
  agent none

  environment {
    CI = true
    DEPLOY_DIR = false
  }

  stages {
    stage('Build') {
      agent { label 'linux' }

      stages {
        // Build > Prepare
        stage('Prepare') {
          steps {
            sh 'whoami'
            sh 'node --version'
            sh 'npm --version'
            sh 'npm config list'
          }
          post {
            always {
              updateGitlabCommitStatus name: 'jenkins-build', state: 'running'
            }
          }
        }

        // Build > Install dependencies
        stage('Install dependencies') {
          steps {
            sh 'npm ci'
          }
        }

        stage('Lint and Build') {
          parallel {
            // Build > Lint
            stage('Lint') {
              steps {
                // catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE')
                catchError {
                  sh 'npm run eslint -- -f checkstyle -o eslint.xml'
                }
              }
              post {
                always {
                  recordIssues enabledForFailure: true, tools: [esLint(id: 'eslint', name: 'ESlint ', pattern: 'eslint.xml')]
                }
              }
            }

            // Build > Build
            stage('Build') {
              steps {
                sh 'npm run build'
              }
            }
          }
        }
      }
    }
    stage('Integration Tests') {
      // Use docker agent for integration test
      agent { label 'docker' }
      environment {
        MONGODB_TEST_URL = 'mongodb://mongoadmin:mongopass@mongodb:27017/feathers-test?authSource=admin'
      }
      steps {
        script {
          docker.image('mongo')
            .withRun('-e "MONGO_INITDB_ROOT_USERNAME=mongoadmin" -e "MONGO_INITDB_ROOT_PASSWORD=mongopass"') {c ->
            docker.image('node:10-alpine').inside("--link ${c.id}:mongodb") {
              sh 'npm run test'
            }
          }
        }
      }
      post {
        success {
          publishHTML target: [
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'coverage/lcov-report',
            reportFiles: 'index.html',
            reportName: 'Coverage Report (NYC)',
            reportTitles: ''
          ]

          cobertura (
            autoUpdateHealth: false,
            autoUpdateStability: false,
            coberturaReportFile: 'coverage/cobertura-coverage.xml',
            conditionalCoverageTargets: '70, 0, 0',
            failNoReports: false,
            failUnhealthy: false,
            failUnstable: false,
            lineCoverageTargets: '80, 0, 0',
            maxNumberOfBuilds: 0,
            methodCoverageTargets: '80, 0, 0',
            sourceEncoding: 'ASCII',
            zoomCoverageChart: false
          )
        }
      }
    }
  }
  post {
    success {
      updateGitlabCommitStatus name: 'jenkins-build', state: 'success'
    }
    failure {
      updateGitlabCommitStatus name: 'jenkins-build', state: 'failed'
    }
  }
}
