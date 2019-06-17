node {
    stage('Preparation') { // for display purposes
        // Get some code from a GitHub repository
        git 'https://github.com/NeerajAgarwal7/Minimal-Todo.git'
      
    }
    
    stage('Code Quality') {
        sh "./gradlew clean"
        withSonarQubeEnv('mySonar') {
            sh "./gradlew sonarqube -Dsonar.host.url=http://18.138.16.110 --stacktrace"
        }
    }
   
    stage("SonarQube Quality Gate") { 
        withSonarQubeEnv('mySonar'){
         	timeout(time: 2, unit: 'MINUTES') { 
                def qg = waitForQualityGate() 
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
            }
        }
    }
   
    stage('Build') {
        //sh "echo $ANDROID_HOME"
        sh "./gradlew clean"
        sh "./gradlew build"
        sh "sudo cp app/build/outputs/apk/debug/*.apk ."
    }
    
    stage('Lint') {
        androidLint canComputeNew: false,
        defaultEncoding: '',
        healthy: '',
        pattern: '**/lint-results.xml',
        unHealthy: ''
    }
   
   stage('Upload'){
    hockeyApp applications: [
    [
        apiToken: '8e4fed5ee7074037950eb4eac94904b0',
        downloadAllowed: true,
        filePath: '*.apk',
        releaseNotesMethod: manual(isMarkdown: true, releaseNotes: 'Bug fixes and minor updates'),
        uploadMethod: versionCreation('753a0389545e41bb83f5efd0e93bf946')
    ]
    ], debugMode: false, failGracefully: false
   }
   
   stage('Slack'){
       slackSend color: 'good', message: 'Click the link to download the app $HOCKEYAPP_INSTALL_URL_0'
   }
   
}
