node {
    stage('Build') {
        withDockerContainer('python:2-alpine') {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }
    }
    stage('Test') {
        withDockerContainer('qnib/pytest') {
            sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            junit 'test-reports/results.xml'
        }
    }
    stage('Manual Approval'){
        input 'Click the Proceed button to deploy the app'
    stage('Deploy') {
        def workspace = pwd()
        withEnv(["""VOLUME=$workspace/sources:/src""", 'IMAGE=cdrx/pyinstaller-linux:python2']) {
            dir(path: env.BUILD_ID) {
                unstash(name: 'compiled-results')
                sh '''docker run --rm -v $VOLUME $IMAGE 'pyinstaller -F add2vals.py' '''
            }
            archiveArtifacts artifacts: "sources/dist/add2vals"
            sh '''docker run --rm -v $VOLUME $IMAGE 'rm -rf build dist' '''
            sleep time: 1, unit: 'MINUTES'
        }
    }
}
