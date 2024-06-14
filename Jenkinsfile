pipeline{
  agent any
  tools{
    maven 'maven-3'
  }

  stages{

    stage('increment version'){
      steps{
        script{
          echo 'incrementing application version.....'
          sh 'mvn build-helper:parse-version versions:set \
            -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} version:commit'
          def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
          def version = matcher[0][1]
          nv.IMAGE_NAME = "$version-$BUILD_NUMBER"
        }
      }
    }
    stage('build jar'){
      steps{
        script{
          echo 'building application artifact'
          sh 'mvn clean package'
        }
      }
    }

    stage('buildImage'){
      steps{
        script{
          echo 'building application into an image'
          withCredentials([usernamePassword(credentialsId:'dockerhub-credentials',usernameVariable:'USER',passwordVariable:'PASS')]){
            sh 'echo ${PASS}|docker login -u ${USER} --password-stdin'
            sh "docker build -t nanaot/java-app:$IMAGE_NAME ."
            sh "docker push nanaot/java-app:$IMAGE_NAME"
          }
        }
      }
    }
  }

}