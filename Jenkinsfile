pipeline {

  environment {
    major=0
    minor=1
    patch="${env.BUILD_ID}"
    name="${env.JOB_NAME}"
    version="$major.$minor.$patch"
    exec_name="datagram-dump-$version"
    publishDir="/var/www/html/$name/$version"
    lastPublishDir="/var/www/html/$name/last"
    binMasterPublishDir="$publishDir/Linux"
    binWinx64Dir="windows-x64"
    binWinx64PublishDir="$publishDir/$binWinx64Dir"
  }

  agent none
  stages {
    stage('TEST MASTER'){
      agent { label 'master'}
      steps {
        sh "make coverage"
        sh "make test"
      }
      post {
        always {
          publishCppcheck pattern:'build/coverage/report/cppcheck.xml'
          step([$class: 'CoberturaPublisher', autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'build/coverage/report/gcovr-report*.html', failUnhealthy: false, failUnstable: false, maxNumberOfBuilds: 0, onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false])
          junit 'build/test-report/*.xml'
        }
      }
    }

    stage('DOCUMENTATION'){
      agent { label 'master'}
      steps {
        sh "make doc"
      }
    }

    stage('BUILD WINDOWS 10 AND TEST'){
      agent { label 'windows10-x64-2'}
      steps {
        bat "Scripts\\change_makefile_name.bat"
        //compile
        bat "make test"
        archiveArtifacts('build\\bin\\datagram-dump.exe')
        archiveArtifacts('build\\bin\\cidco-decoder.exe')
        archiveArtifacts('build\\bin\\datagram-list.exe')
        archiveArtifacts('build\\bin\\georeference.exe')

      }
      post {
        always {
          junit 'build\\test-report\\*.xml'
        }
      }
    }

    stage('BUILD MASTER'){
      agent { label 'master'}
      steps {
        sh 'make'
      }
    }

    stage('PUBLISH ON SERVER'){
      agent { label 'master'}
      steps {
        sh 'mkdir -p $binMasterPublishDir'
        sh 'mkdir -p $binWinx64PublishDir'
        sh 'cp -r build/bin/datagram-dump $binMasterPublishDir/$exec_name'
        sh 'cp -r build/bin/cidco-decoder $binMasterPublishDir/cidco-decoder-$version'
        sh 'cp -r build/bin/datagram-list $binMasterPublishDir/datagram-list-$version'
        sh 'cp -r build/bin/georeference $binMasterPublishDir/georeference-$version'
        sh 'cp  /var/lib/jenkins/jobs/$name/builds/$patch/archive/build/bin/datagram-dump.exe  $binWinx64PublishDir/$exec_name.exe'
        sh 'cp  /var/lib/jenkins/jobs/$name/builds/$patch/archive/build/bin/cidco-decoder.exe  $binWinx64PublishDir/cidco-decoder-$version.exe'
        sh 'cp  /var/lib/jenkins/jobs/$name/builds/$patch/archive/build/bin/datagram-list.exe  $binWinx64PublishDir/datagram-list-$version.exe'
        sh 'cp  /var/lib/jenkins/jobs/$name/builds/$patch/archive/build/bin/georeference.exe  $binWinx64PublishDir/georeference-$version.exe'
      }
    }
  }

}
