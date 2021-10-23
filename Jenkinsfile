pipeline {
    agent any

    tools {
        maven 'Maven'
        nodejs 'NodeJs'
    }
  
    stages {
        stage('initial'){
            steps{
                figlet 'Inital'
                
             sh '''
              echo "PATH = ${PATH}"
              echo "M2_HOME = ${M2_HOME}"
              '''
            }
        }

        stage('Compile'){
            steps{
                figlet 'Compile'
                sh 'mvn clean compile -e'
            }
        }
        
        stage('Test'){
            steps{
                figlet 'Test'
                sh 'mvn clean test -e'
            }
        }
        
        stage('SCA'){
            steps{
                figlet 'Dependency-Check'
                sh 'mvn org.owasp:dependency-check-maven:check'
                
                archiveArtifacts artifacts: 'target/dependency-check-report.html', followSymlinks: false
            }
        }
        
        stage('Sonarqube'){
           steps{
               figlet 'SonarQube'
               script{
                   def scannerHome = tool 'Sonar Scanner'
                   //def scannerHome = tool 'Sonar Server'
                   
                   withSonarQubeEnv('Sonar Server'){
                       sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=ms-maven -Dsonar.sources=. -Dsonar.projectBaseDir=${env.WORKSPACE} -Dsonar.java.binaries=target/classes -Dsonar.exclusions='**/*/test/**/*, **/*/acceptance-test/**/*, **/*.html'"
                   }
               }
               
           
        
          stage('DAST') {
            steps {
                script {
                    try {
                        echo "Inicio de Scanneo Dinamico"
                        sh "docker exec zap zap-cli --verbose quick-scan http://zero.webappsecurity.com:8090 -l Medium" 
                        //sh "docker exec zap zap-cli --verbose alerts --alert-level Medium -f json | jq length"
                        currentBuild.result = 'SUCCESS' 
                    }
                    catch (Exception e) {
                            //echo e.getMessage() 
                            //currentBuild.result = 'FAILURE'
                            println ("Revisar Reporte ZAP. Se encontraron Vulnerabilidades.")

                        }
                    }  
                    echo currentBuild.result 
                    echo "Generacion de Reporte"
                    sh "docker exec zap zap-cli --verbose report -o /zap/reports/owasp-quick-scan-report.html --output-format html"
                    publishHTML target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: '/var/jenkins_home',
                        reportFiles: 'owasp-quick-scan-report.html',
                        reportName: 'Analisis DAST'
                      ]          
            }
        }
        		
            
        
                 
           
