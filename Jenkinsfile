pipeline {
  agent any
  parameters {
      choice(
          choices: 'incremental\nclean',
          description: 'Build type',
          name: 'BUILD_ACTION')
  }
  stages {
    stage('Checkout') {
      steps {
        script {
          if( params.BUILD_ACTION == 'clean' ) {
              echo 'Preperation for VECTOR Clean build'
              sh 'rm -rf vysionics_bsp'
              checkout([$class: 'SubversionSCM', 
              additionalCredentials: [], 
              excludedCommitMessages: '', 
              excludedRegions: '', 
              excludedRevprop: '', 
              excludedUsers: '', 
              filterChangelog: false, 
              ignoreDirPropChanges: false, 
              includedRegions: '', 
              locations: [[credentialsId: 'vys-svn', 
                          depthOption: 'infinity', 
                          ignoreExternalsOption: true, 
                          local: 'vysionics_bsp', 
                          remote: 'svn://vys-svn/vysionics_bsp/trunk']], 
              workspaceUpdater: [$class: 'UpdateUpdater']])        
          }
          else {
            dir('./vysionics_bsp/vector_incremental_build/build/vysionics-HEAD') {
              checkout([$class: 'SubversionSCM', 
              additionalCredentials: [], 
              excludedCommitMessages: '', 
              excludedRegions: '', 
              excludedRevprop: '', 
              excludedUsers: '', 
              filterChangelog: false, 
              ignoreDirPropChanges: false, 
              includedRegions: '', 
              locations: [[credentialsId: 'vys-svn', 
                          depthOption: 'infinity', 
                          ignoreExternalsOption: true, 
                          local: './', 
                          remote: 'svn://vys-svn/specs_3_vector/Software/trunk']], 
              workspaceUpdater: [$class: 'UpdateUpdater']])
              checkout([$class: 'SubversionSCM', 
              additionalCredentials: [], 
              excludedCommitMessages: '', 
              excludedRegions: '', 
              excludedRevprop: '', 
              excludedUsers: '', 
              filterChangelog: false, 
              ignoreDirPropChanges: false, 
              includedRegions: '', 
              locations: [[credentialsId: 'vys-svn', 
                          depthOption: 'infinity', 
                          ignoreExternalsOption: true, 
                          local: './ipf', 
                          remote: 'svn://vys-svn/ipf/trunk']], 
              workspaceUpdater: [$class: 'UpdateUpdater']])
              checkout([$class: 'SubversionSCM', 
              additionalCredentials: [], 
              excludedCommitMessages: '', 
              excludedRegions: '', 
              excludedRevprop: '', 
              excludedUsers: '', 
              filterChangelog: false, 
              ignoreDirPropChanges: false, 
              includedRegions: '', 
              locations: [[credentialsId: 'vys-svn', 
                          depthOption: 'infinity', 
                          ignoreExternalsOption: true, 
                          local: './libVysUtils', 
                          remote: 'svn://vys-svn/vysionics_common/trunk/libVysUtils']], 
              workspaceUpdater: [$class: 'UpdateUpdater']])
              checkout([$class: 'SubversionSCM', 
              additionalCredentials: [], 
              excludedCommitMessages: '', 
              excludedRegions: '', 
              excludedRevprop: '', 
              excludedUsers: '', 
              filterChangelog: false, 
              ignoreDirPropChanges: false, 
              includedRegions: '', 
              locations: [[credentialsId: 'vys-svn', 
                          depthOption: 'infinity', 
                          ignoreExternalsOption: true, 
                          local: './libVysCrypt', 
                          remote: 'svn://vys-svn/vysionics_common/trunk/libVysCrypt']], 
              workspaceUpdater: [$class: 'UpdateUpdater']])
              sh 'rm -f .stamp_configured .stamp_built .stamp_staging_installed .stamp_target_installed'
              sh 'rm -rf buildroot-build'
            }
            dir('./vysionics_bsp/vector_incremental_build') {
              sh 'rm -f images/*.xz'
            }
          }
        }
      }
    }
    stage('Build') {
      steps {
        script {
          if( params.BUILD_ACTION == 'clean' ) {
            echo 'Build VECTOR_defconfig'
            wrap([$class: 'TimestamperBuildWrapper']) {
              sh 'mkdir -p ./vysionics_bsp/vector_incremental_build'
              dir('./vysionics_bsp/vector_incremental_build') {
                  sh 'mkdir -p target/usr/vysionics/etc/'
                  sh 'touch target/usr/vysionics/etc/md5sums.txt'
                  sh 'make BR2_JLEVEL=0 BR2_EXTERNAL=../vys_buildroot O=$PWD -C../buildroot/ VECTOR_defconfig'
                  sh 'make BR2_BUILD_TESTS=y'
                  //sh 'make'
              }
            }
          }
          else {
            echo 'Build Incremental of vysionics-HEAD'
            wrap([$class: 'TimestamperBuildWrapper']) {
              dir('./vysionics_bsp/vector_incremental_build') {
                  sh 'make BR2_BUILD_TESTS=y'            
              }
            }
          }
        }
      }
    }
    stage('Test') {
      parallel {
        stage('Unit Test') {
          environment { 
            LD_LIBRARY_PATH = "$WORKSPACE/vysionics_bsp/vector_incremental_build/target/usr/vysionics/lib/:$WORKSPACE/vysionics_bsp/vector_incremental_build/target/usr/lib/"
          }
          steps {
            dir('./vysionics_bsp/vector_incremental_build/build/vysionics-HEAD/buildroot-build/') {
              dir('./aspd/src/aspd-build'){
                sh 'LD_LIBRARY_PATH=${LD_LIBRARY_PATH} ./src/test/test_aspd --gtest_output=xml:test_aspd.xml'
              }
              dir('./bofservice/src/bofservice-build'){
                sh 'LD_LIBRARY_PATH=${LD_LIBRARY_PATH} ./src/test/test_bofservice --gtest_output=xml:test_bofservice.xml'
              }
              // dir('./commissioning/src/commissioning-build'){
              //  sh 'LD_LIBRARY_PATH=${LD_LIBRARY_PATH} ./src/test/test_commissioning --gtest_output=xml:test_commissioning.xml'
              //}
              dir('./libVysUtils/src/libVysUtils-build'){
                sh 'LD_LIBRARY_PATH=${LD_LIBRARY_PATH} ./src/test/test_VysUtils --gtest_output=xml:test_VysUtils.xml'
              }
              // dir('./libengine/src/libengine-build'){
              //   sh 'LD_LIBRARY_PATH=${LD_LIBRARY_PATH} ./src/test/test_libengine --gtest_output=xml:test_libengine.xml'
              // }
              dir('./libengine_qfree/src/libengine_qfree-build'){
                sh 'LD_LIBRARY_PATH=${LD_LIBRARY_PATH} ./src/test/test_libengineq --gtest_output=xml:test_libengineq.xml'
              }
              dir('./nrsd/src/nrsd-build'){
                sh 'LD_LIBRARY_PATH=${LD_LIBRARY_PATH} ./src/test/test_nrsd --gtest_output=xml:test_nrsd.xml'
              }
              dir('./supervisor/src/supervisor-build'){
                sh 'LD_LIBRARY_PATH=${LD_LIBRARY_PATH} ./test/test_supervisor --gtest_output=xml:test_supervisor.xml'
              }
            }
          }
          post {
            always {
                step([$class: 'XUnitPublisher', testTimeMargin: '3000', thresholdMode: 1, thresholds: [[$class: 'FailedThreshold', failureNewThreshold: '', failureThreshold: '', unstableNewThreshold: '', unstableThreshold: ''], [$class: 'SkippedThreshold', failureNewThreshold: '', failureThreshold: '', unstableNewThreshold: '', unstableThreshold: '']], tools: [[$class: 'GoogleTestType', deleteOutputFiles: false, failIfNotNew: true, pattern: '**/src/*-build/test_*.xml', skipNoTestFiles: false, stopProcessingIfError: true]]])
            }
          }
        }
        stage('Static Analysis') {
          steps {
            echo 'Static analysis'
          }
        }
      }
    }
    stage('Package & Deploy') {
      steps {
        echo 'Package and Deploy to VECTOR(s)'
        wrap([$class: 'TimestamperBuildWrapper']) {
          dir('./vysionics_bsp/vector_incremental_build') {
            echo 'Call ./installer.sh'  
            sh './installer.sh'
            sh 'installer/x86_64/vector_update_x86_64.run	-a 192.168.1.201'
            archiveArtifacts artifacts: 'images/*bzImage'
            archiveArtifacts artifacts: 'images/*rootfs.cpio.xz'

            script{
              def kernel = findFiles glob: 'images/*bzImage'
              echo """${kernel[0].name} ${kernel[0].path} ${kernel[0].directory} ${kernel.length} ${kernel[0].lastModified}"""
                
              def rootfs = findFiles glob: 'images/*rootfs.cpio.xz'
              echo """${rootfs[0].name} ${rootfs[0].path} ${rootfs[0].directory} ${rootfs.length} ${rootfs[0].lastModified}"""
              def timestamp = """${rootfs[0].name}""".split( '_' )
              def server = Artifactory.server('sandbox-server')
              def uploadSpec = """{
                  "files": [
                      {
                          "pattern": "images/*bzImage",
                          "target": "VECTOR/build/incremental/${timestamp}/",
                          "props": "p1=v1;p2=v2"
                      },
                      {
                         "pattern": "images/*rootfs.cpio.xz",
                          "target": "VECTOR/build/incremental/${timestamp}/",
                          "props": "p1=v1;p2=v2"
                      }
                  ]
              }"""
              // Upload files to Artifactory:
              def buildInfo = Artifactory.newBuildInfo()
              buildInfo.env.capture = true
              server.upload(uploadSpec, buildInfo)
            }
            // script {
            //     def kernel = findFiles glob: 'images/*bzImage'
            //     echo """${kernel[0].name} ${kernel[0].path} ${kernel[0].directory} ${kernel.length} ${kernel[0].lastModified}"""
              
            //     def rootfs = findFiles glob: 'images/*rootfs.cpio.xz'
            //     echo """${rootfs[0].name} ${rootfs[0].path} ${rootfs[0].directory} ${rootfs.length} ${rootfs[0].lastModified}"""
              
            //     def timestamp = """${rootfs[0].name}""".split( '_' )
            //     def version = '2.6.0'

            //     // sh """curl -v -F --user \'admin:admin123\' --upload-file ${rootfs[0].path} http://10.125.16.44:8082/vector/standard/incremental/${env.BUILD_NUMBER}/${rootfs[0].name}"""
            //     // sh """curl -v -F --user \'admin:admin123\' --upload-file ${kernel[0].path} http://10.125.16.44:8082/vector/standard/incremental/${env.BUILD_NUMBER}/${kernel[0].name}"""

            //   nexusArtifactUploader(
            //     nexusVersion: 'nexus3',
            //     protocol: 'http',
            //     nexusUrl: '10.125.16.44:8082',
            //     groupId: 'incremental.build',
            //     version: timestamp[0],
            //     repository: 'VECTOR/',
            //     credentialsId: 'nexus-jenkins',
            //     artifacts: [
            //       [
            //          artifactId: """${rootfs[0].name}""",
            //          classifier: '',
            //          file: """${rootfs[0].path}""",
            //          type: 'xz'
            //       ],                  
            //       [
            //          artifactId: """${kernel[0].name}""",
            //          classifier: '',
            //          file: """${kernel[0].path}""",
            //          type: 'bzImage'
            //       ]
            //     ]
            //  )
              // nexusArtifactUploader {
              //   nexusVersion('nexus3')
              //   protocol('http')
              //   nexusUrl('http://10.125.16.44:8082')
              //   groupId('jenoptik.uk')
              //   version('2.6.0')
              //   repository('vector-incremental-snapshots')
              //   credentialsId('nexus-jenkins')
              //   artifact {
              //       artifactId('rootfs')
              //       type('xz')
              //       classifier('snapshot')
              //       file('${rootfs[0].path}')
              //   }
              //   artifact {
              //       artifactId('kernel')
              //       type('bzImage')
              //       classifier('snapshot')
              //       file('${kernel[0].path}')
              //   }
              // }
            //}
          }
        }
      }
    }
    stage('Smoke Tests') {
      steps {
        echo 'Smoke tests to verify deployment(s)'
      }
    }
    stage('Integration Tests') {
      steps {
        echo 'Integration tests on target'
      }
    }
    stage('Finalise') {
      steps {
        echo 'Promote build to QA'
      }
    }
  }
}
