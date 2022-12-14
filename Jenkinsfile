pipeline {                                                                           
  agent any
  tools {
        maven "mvn3.8.6"
  }
  environment {
    NEXUS_VERSION = "nexus3"
    NEXUS_PROTOCOL = "http"
    NEXUS_URL = "192.168.1.24:8081"
    NEXUS_REPOSITORY = "vprofile-repo"
    NEXUS_REPO_ID    = "vprofile-repo"
    NEXUS_CREDENTIAL_ID = "nexuslogin"
    ARTID = "vprofile"
    ARTVERSION = "${env.BUILD_ID}"
    ART_PATH = "target/vprofile-v2.war"
    ARTTYPE = "war"
  }

  stages{                                                                            
    stage('Fetch Code'){                                                             
      steps{                                                                         
        git branch: 'main', url:'https://github.com/darkrune0/vprofile-PaaC.git'
      }                                                                              
    }                                                                                
    stage('build'){                                                                  
      steps{                                                                         
        sh 'mvn clean install -DskipTests'                                                             
      }
      post{
        success{
            echo "Now archiving."
            //archiveArtifacts artifacts: '**/*.war'
        }
      }                                                                      
    }                                                                                
    stage('Test'){                                                                   
      steps{                                                                         
        sh 'mvn test'                                                                
      }                                                                              
    }
    stage('checkstyle Analysis'){
        steps{
            sh 'mvn checkstyle:checkstyle'
        }
    }
    //this stage calls for a sonarqube report
    stage('Sonar Analysis'){
        //env variable
        environment{
            scannerHome = tool 'sonar4.7'
        }
        steps{
            withSonarQubeEnv('sonar'){
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                  
                '''
            }
        }
    }
    //this step makes jenkins wait for sonarqube report
    stage('Quality Gate 666'){
        steps{
            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
        }
    }
    
    
    stage("Publish to Nexus Repository Manager") {
        steps {
            script {
                // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                pom = readMavenPom file: "pom.xml";

                // Find built artifact under target folder
                filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                
                // Print some info from the artifact found
                echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"

                // Extract the path from the File found    
                artifactPath = filesByGlob[0].path;

                // Assign to a boolean response verifying If the artifact name exists
                artifactExists = fileExists artifactPath;

                if(artifactExists) {
                    echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: 'qa',
                        version: ARTVERSION,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            //uploads the war file
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: artifactPath,
                            type: pom.packaging],

                            //uploads the pom file for more info
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: "pom.xml",
                            type: "pom"]
                        ]
                    );
                } 
                else {
                    error "*** File: ${artifactPath}, could not be found";
                }

                // echo "env variables #############################";
                // echo NEXUS_VERSION;
                // echo NEXUS_PROTOCOL;
                // echo NEXUS_URL;
                // echo pom.groupId;
                // echo ARTVERSION;
                // echo NEXUS_REPOSITORY;
                // echo NEXUS_CREDENTIAL_ID;

                // echo pom.artifactId;
                // echo artifactPath;
                // echo pom.packaging;
            }
        }
    }


  }                                                                                  
}                                                                                    
