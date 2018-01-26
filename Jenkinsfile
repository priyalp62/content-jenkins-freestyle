pipeline {
  agent none

  environment {
    MAJOR_VERSION = 1
  }

  stages {
    stage('Unit Tests') {
      agent {
        label 'apache'
      }
      steps {
         sh 'ant -f test.xml -v'
         junit 'reports/result.xml'
      }
    }
   stage('build') {
     agent {
       label 'apache'
     }
     steps {
      sh 'ant -f build.xml -v'
   }
   post {
     success {
     archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
     }
   }
   }
   stage('deploy') {
     agent {
       label 'apache'
     }
     steps {
       sh "if ![ -d '/var/www/html/rectangle/all/${env.BRANCH_NAME}' ]; then mkdir /var/www/html/rectangle/all/${env.BRANCH_NAME}; fi"
       sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangle/all/${env.BRANCH_NAME}/"
     }
   }
   stage("Running on centos"){
     agent {
       label 'centos'
     }
     steps {
       sh "wget http://priyalp621.mylabserver.com/rectangle/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
       sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
     }
   }
   stage("test on Debian"){
     agent {
       docker 'openjdk:8u121-jre'
     }
     steps {
       sh "wget http://priyalp621.mylabserver.com/rectangle/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
       sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
     }
   }
   stage('promote to green') {
     agent {
       label 'apache'
     }
     when {
       branch 'master'
     }
     steps {
       sh "cp /var/www/html/rectangle/all/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangle/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
     }
   }
   stage('promote development branch to master') {
   agent {
     label 'apache'
   }
   when {
     branch 'development'
   }
   steps {
     echo "stashing any local changes"
     sh 'git stash'
     echo "checking out development"
     sh 'git checkout development'
     echo 'checking out the master branch'
     sh 'git pull origin'
     sh 'git checkout master'
     echo 'Merging development into master branch'
     sh 'git merge development'
     echo 'pushing to origin master'
     sh 'git push origin master'
     echo 'tagging the release'
     sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
     sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
   }
  }
 }
}
