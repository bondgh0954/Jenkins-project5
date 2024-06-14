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
            -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} versions:commit'
          def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
          def version = matcher[0][1]
          env.IMAGE_NAME = "$version-$BUILD_NUMBER"
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
            sh 'echo $PASS|docker login -u $USER --password-stdin'
            sh "docker build -t nanaot/java-app:${IMAGE_NAME} ."
            sh "docker push nanaot/java-app:${IMAGE_NAME}"
          }
        }
      }
    }
    stage('deploy'){
      steps{
        script{
          echo 'deploying application'
        }
      }
    }

    stage('commit update'){
      steps{
        script{
          echo 'commiting update version to git repository'
          withCredentials([usernamePassword(credentialsId:'github-credentials',usernameVariable:'USER',passwordVariable:'PASS')]){

            sh 'git config --global user.email "jenkins@example.com" '
            sh 'git config --global user.name "jenkins"'

            sh "git remote set-url origin https://${USER}:${PASS}@git@github.com/bondgh0954/Jenkins-project5.git"
            sh 'git add .'
            sh 'git commit -m "update version"'
            sh 'git push origin HEAD:master'
          }
        }
      }
    }
  }

}