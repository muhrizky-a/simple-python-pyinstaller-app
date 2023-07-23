node {
    docker.image('python:2-alpine').inside() {
        stage('Build') {
            sh 'python -m py_compile sources/add2vals.py sources/calc.py'
        }
    }
    docker.image('qnib/pytest').inside() {
        try {
            stage('Test') {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
        } catch (e) {
            echo 'This will run only if failed'
            throw e
        } finally {
            junit 'test-reports/results.xml'
        }
    }
    stage('Deploy') {
        try {
            env.VOLUME = "${pwd()}/sources:/src"
            env.IMAGE = 'cdrx/pyinstaller-linux:python2'

            dir(env.BUILD_ID) {
                sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'pyinstaller -F add2vals.py'"
            }

        } catch (e) {
            echo 'This will run only if failed'	
            throw e
        } finally {
	    if (currentBuild.result == 'SUCCESS') {
                archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'rm -rf build dist'"
            }
        }
    }
}
