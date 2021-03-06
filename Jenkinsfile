//Version - 1.1
node {
    stage('SCM Checkout') {
        git 'https://github.com/deepakrohan1/DevOps-Demo-WebApp.git'
    }
    
     stage('Compile Package') {
        def mvnHome = tool name: 'maven3.6', type: 'maven'
        sh "${mvnHome}/bin/mvn clean package"
        // slackSend channel: 'general', message: 'Packaging Java Code'
    }
    
    // stage('Sonarqube Analysis') {
    //      def runner = tool name: 'sonarqube', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
    //      withSonarQubeEnv('sonarqube') {
    //          sh "${runner}/bin/sonar-scanner"
    //         //  slackSend channel: 'alerts-fromjenkins', message: 'Sonar Scan completed'
    //     }
    // }
    
    stage('Artifactory Push') {
        def server = Artifactory.server "artifactory"
        def rtMaven = Artifactory.newMavenBuild()
        def buildInfo
        rtMaven.tool = "maven3.6"
        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server
        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
        server.publishBuildInfo buildInfo
    }
    
    stage('Deploy File to Test') {
        deploy adapters: [tomcat8(credentialsId: 'tomcat-aws', path: '', url: 'http://3.141.174.85:8080/')], contextPath: '/QAWebapp', onFailure: false, war: '**/*.war'
    }
    

     stage('Sanity Check') {
        def workspace = pwd()
        print("{$workspace}")
         def mvnHome = tool name: 'maven3.6', type: 'maven'
        sh "${mvnHome}/bin/mvn -f $workspace/functionaltest/pom.xml test"
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
      //  slackSend channel: 'alerts-fromjenkins', message: 'Completed the Sanity Check Review reports under sure-fire'
    }
     stage('Deploy File to Prod') {
        deploy adapters: [tomcat8(credentialsId: 'tomcat-aws', path: '', url: 'http://3.141.174.85:8080/')], contextPath: '/ProdWebapp', onFailure: false, war: '**/*.war'
    }
    stage('Docker push') {
        withDockerRegistry(credentialsId: 'DOCKER_HUB_PWD', toolName: 'dockerhub', url: 'https://registry.hub.docker.com/') {
        def image = docker.build("kaoster/apps:${env.BUILD_ID}")
        image.push()
    }
    }
    
     //  stage('Blazemeter tests'){
     //   blazeMeterTest credentialsId: 'blazemeter', testId: '9014496.taurus', workspaceId: '756008'
    //}
    
    
}
