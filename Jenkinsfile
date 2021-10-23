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
               
           
        
           stage('DAST'){
            steps{
                figlet 'Owasp Zap DAST'
        		
        		script{
        		   env.DOCKER = tool "Docker"
        		   env.DOCKER_EXEC = "${DOCKER}/bin/docker"
        		   env.TARGET = 'http://zero.webappsecurity.com'
        	
        		    sh '${DOCKER_EXEC} rm -f zap2'
        		    sh "${DOCKER_EXEC} pull owasp/zap2docker-stable"
                    sh '${DOCKER_EXEC} run --add-host="localhost:192.168.1.110" --rm -e LC_ALL=C.UTF-8 -e LANG=C.UTF-8 --name zap2 -u zap -p 8081:8081 -d owasp/zap2docker-stable zap.sh -daemon -port 8081 -host 0.0.0.0 -config api.disablekey=true'
                    
sh '${DOCKER_EXEC} run --user $(id -w):$(id -q) --add-host="localhost:192.168.1.110" -v /root/jenkins/tools:/zap/wrk/:rw --rm -i owasp/zap2docker-stable zap-baseline.py -t "http://zero.webappsecurity.com" -I -r zap_baseline_report.html -l PASS'	
        		   
        		   publishHTML([
        				    allowMissing: false,
        				    alwaysLinkToLastBuild: false,
        				    keepAll: false,
        				    reportDir: '/var/jenkins_home/tools/',
        				    reportFiles: 'zap_baseline_report.html',
        				    reportName: 'HTML Report',
        				    reportTitles: ''])
        		
            
        
                 
           }
          }
           }
           }
        }
    }
       
    
