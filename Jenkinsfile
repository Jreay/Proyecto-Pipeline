pipeline {
  agent any

  tools {
    jdk 'jdk-11'
    maven 'mvn-3.6.3'
  }

  stages {
    stage('Build') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh "mvn package"
        }
      }
    }

    stage ('OWASP Dependency-Check Vulnerabilities') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh 'mvn dependency-check:check'
        }

        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      }
    }

    stage ('PMD SpotBugs') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh 'mvn pmd:pmd pmd:cpd spotbugs:spotbugs'
        }

        recordIssues enabledForFailure: true, tool: spotBugs()
        recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml')
        recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
      }
    }

    stage ('ZAP') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh 'mvn zap:analyze'
          publishHTML (target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'target/zap-reports',
                reportFiles: 'zapReport.html',
                reportName: "ZAP report"
              ])
        }
      }
    }

    stage('SonarQube analysis') {
      steps {
        withSonarQubeEnv(credentialsId: 'sonarqube-secret', installationName: 'sonarqube-server') {
          withMaven(maven : 'mvn-3.6.3') {
            sh 'mvn sonar:sonar -Dsonar.host.url=http://sonarqube-server:9000 -Dsonar.login=squ_71a13e966896f298ae4b2dccf0e2cfe8ef58fe66'
          }
        }
      }
    }
  }
}
