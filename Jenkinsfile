stage('CI')
{
    node {
    
	// this grabs code from where scm is configured in job
	checkout scm

        //git branch: 'jenkins2-course', 
        //    url: 'https://github.com/mcknz/solitaire-systemjs-course'
    
        // pull dependencies from npm
        // on windows use: bat 'npm install'
        sh 'npm install'
    
        // stash code & dependencies to expedite subsequent testing
        // and ensure same code & dependencies are used throughout the pipeline
        // stash is a temporary archive
        stash name: 'everything', 
              excludes: 'test-results/**', 
              includes: '**'
        
        // test with PhantomJS for "fast" "generic" results
        // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
        sh 'npm run test-single-run -- --browsers PhantomJS'
        
        // archive karma test results (karma is configured to export junit xml files)
        step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
    }

    // demo a second agent, verify unstash works    
    node('ubuntu') {
        sh 'ls'
        sh 'rm -rf *'
        unstash 'everything'
        sh 'ls'
    }
}

stage('Browser Testing') {
    parallel chrome: {
        runTests("Chrome")
    }, firefoxTests: {
        runTests("Firefox")
    },
    //  safari {
    //    runTests("Safari")  
    //},
    failFast: true
}

def runTests(browser) {
    node {
        sh 'rm -rf *'
        unstash 'everything'
        sh "npm run test-single-run -- --browsers ${browser}"
        junit 'test-results/**/test-results.xml'
    }
}

// needs a node allocation
node {
    notify("Deploy to staging?")
}

// do not allocate to node b/c it is blocking --
//   runs on a flyweight executor
input 'Deploy to staging?'

stage('Deploy') {
    // write build number to index page
    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    
    // deploy to docker container
    // sh 'docker compose up -d --build'
    
    notify 'Software deployed!'
}


def notify(status) {
    emailext (
      to: "mail@mcknz.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
