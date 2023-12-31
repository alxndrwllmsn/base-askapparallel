def get_email() {
   dir(path: "${env.REPO}" ) {
      return sh (script:"git log -1 --pretty=format:'%ae'",returnStdout:true).trim()
   }
}  
pipeline {
  agent {
    docker {
      image 'sord/devops:lofar'
    }

  }
  stages {
      stage('Building Dependency (ASKAP)') {
      steps {
        dir(path: '.') {
          sh '''if [ -d base-askap ]; then
echo "base-askap directory already exists"
rm -rf base-askap
fi
git clone https://bitbucket.csiro.au/scm/askapsdp/base-askap.git
cd base-askap
git checkout develop
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=${PREFIX} ../
make -j2
make -j2 install
'''
        }
      }
      }

      stage('Building Debug') {
      steps {
        dir(path: '.') {
          sh '''git fetch --tags
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=${PREFIX} -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-coverage" ../
make 
'''
        }
      }
    }

    stage('Building Release') {
      steps {
        dir(path: '.') {
          sh '''git fetch --tags
mkdir build-release
cd build-release
cmake -DCMAKE_INSTALL_PREFIX=${PREFIX} -DCMAKE_BUILD_TYPE=Release ../
make 
'''
        }
      }
    }    
  }

post {
        success {        
             mail to: "${env.EMAIL_TO}",
             from: "jenkins@csiro.au",
             subject: "Succeeded Pipeline: ${currentBuild.fullDisplayName}",
             body: "Build ${env.BUILD_URL} succeeded"
 
        }

        failure {        
             mail to: "${env.EMAIL_TO}",
             from: "jenkins@csiro.au",
             subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
             body: "Something is wrong with ${env.BUILD_URL}"
 
        }
        unstable {
             mail to: "${env.EMAIL_TO}",
             from: "jenkins@csiro.au",
             subject: "Unstable Pipeline: ${currentBuild.fullDisplayName}",
             body: "${env.BUILD_URL} unstable"
        } 
        changed {
             mail to: "${env.EMAIL_TO}",
             from: "jenkins@csiro.au",
             subject: "Changed Pipeline: ${currentBuild.fullDisplayName}",
             body: "${env.BUILD_URL} changed"
        }
 }
 
  environment {
    
    WORKSPACE = pwd()
    PREFIX = "${WORKSPACE}/install"
    REPO = "${WORKSPACE}/base-askapparallel/"
    EMAIL_TO = get_email()

  }
}

