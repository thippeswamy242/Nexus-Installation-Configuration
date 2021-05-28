node{
    stage("Git CheckOut"){
        git url: 'https://github.com/thippeswamy242/Jenkins-Docker.git',branch: 'main'
    }
    
    stage(" Maven Clean Package"){
      def mavenHome =  tool name: "Maven", type: "maven"
      def mavenCMD = "${mavenHome}/bin/mvn"
      sh "${mavenCMD} clean package"
      }
	  
	  stage(" SonarQube analysis"){
      def mavenHome =  tool name: "Maven", type: "maven"
      def mavenCMD = "${mavenHome}/bin/mvn"
	  withSonarQubeEnv('My SonarQube Server'){
      sh "${mavenCMD} sonar:sonar"
      }
	  }
	  
	  stage("Quality Gate"){
  timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
    if (qg.status != 'OK') {
      error "Pipeline aborted due to quality gate failure: ${qg.status}"
    }
	  }
	  }
	  stage('Upload war to Nexus') {
    steps {
      script {
        def mavenPom = readMavenPom file: 'pom.xml'
        nexusArtifactUploader artifacts: [
            [
              artifactId: 'simple-app',
              classifier: '',
              file: "target/simple-app-${mavenPom.version}.war",
              type: 'war'
            ]
          ],
          credentialsId: 'nexus3',
          groupId: 'in.javahome',
          nexusurl: '172.21.14.301:8081',
          nexusVersion: 'nexus3'
        protocol: 'http',
          repositry: 'demoapp-release',
          version: "${mavenPom.version}"

      }
	  
	  
	  }
	  }
	  }
