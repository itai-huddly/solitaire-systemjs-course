stage 'CI'
node (){
	
    checkout scm

    // pull dependencies from npm
    sh 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
}

node (){
    sh 'ls'
    sh 'rm -rf *'
    unstash 'everything'
    sh 'ls'
}

// Parallel integration testing
stage ('Browser Testing'){
    parallel /*Chrome: {
        runTests("Chrome")
    }, */Firefox: {
        //runTests("Firefox")
    }, PhantomJS: {
        runTests("PhantomJS")
    }
}

def runTests(browser){
    node{
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        step([
                $class: 'JUnitResultArchiver',
                testResults: 'test-results/**/test-results.xml'
            ])
    }
}

stage('Input'){
    input message: 'Deploy?', ok: 'Fuck yeah!'
}

stage name: 'Deploy', concurrency: 1
node {
    sh 'echo "$USER"'
    
    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    
    sh 'docker-compose up -d --build'
}



def notify(status){
    emailext (
      to: "wesmdemos@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
