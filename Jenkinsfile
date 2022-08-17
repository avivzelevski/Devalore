pipeline {
    environment {
   
    }

    agent any
        
  stages {
    
    stage('Cloning our Git') {
                steps {
                git 'git@github.com:avivzelevski/Devalore.git'
                }
     }
    
        // Build > Install dependencies
    stage('Install dependencies') {
          steps {
            sh 'npm ci'
            sh './node_modules/.bin/eslint ./'
          }
        }

    stage('Code Lint') {
        
        steps {     
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

    stage('Build') {
              steps {
                sh 'nodejs server.js'
              }
    }
    }

      
    stage('Integration Tests') {
      // Use docker agent for integration test
      agent { label 'docker' }
      
      steps {
        script {
          
              sh 'npm run test'
            }
          }
        }
}
        
  post {
    success {
        setBuildStatus("Build succeeded", "SUCCESS");
    }
    failure {
        setBuildStatus("Build failed", "FAILURE");
    }
  }
}

