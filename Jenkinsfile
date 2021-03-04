node {
    stage('SCM Checkout') {
        git 'https://github.com/deepakrohan1/DevOps-Demo-WebApp.git'
    }

    stage('Compile Package') {
        def mvnHome = tool name: 'Maven3.6.3', type: 'maven'
        sh "${mvnHome}/bin/mvn clean package -DskipTests=true"
        slackSend channel: 'alerts-fromjenkins', message: 'Packaging Java Code'
    }
  
    stage('Sonarqube Analysis') {
         def runner = tool name: 'sonarqube', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
         withSonarQubeEnv('sonarqube') {
             sh "${runner}/bin/sonar-scanner"
             slackSend channel: 'alerts-fromjenkins', message: 'Sonar Scan completed'
        }
    }
     stage('Blazemeter tests'){
        blazeMeterTest credentialsId: 'blazemeter', testId: '9014496.taurus', workspaceId: '756008'
    }

    stage('Deploy File to Test') {
        deploy adapters: [tomcat8(credentialsId: 'tomcat-1', path: '', url: 'http://3.138.156.237:8080/')], contextPath: '/QAWebapp', onFailure: false, war: '**/*.war'
    }

    // Need to install docker pipeline plugins
    stage('Docker push') {
        withDockerRegistry(credentialsId: 'dockerhub', toolName: 'dockerhub', url: 'https://registry.hub.docker.com/') {
        def image = docker.build("deepakrohan/myapp:1")
        image.push()
    }

     stage('Sanity Check') {
        def workspace = pwd()
        print("{$workspace}")
         def mvnHome = tool name: 'Maven3.6.3', type: 'maven'
        sh "${mvnHome}/bin/mvn -f $workspace/Acceptancetest/pom.xml test"
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
        slackSend channel: 'alerts-fromjenkins', message: 'Completed the Sanity Check Review reports under sure-fire'
    }
}
