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
	    if (currentBuild.currentResult == 'SUCCESS') {
                input message: 'Lanjutkan ke tahap Deploy?'
            }
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
	    if (currentBuild.currentResult == 'SUCCESS') {
	        timeout(time: 15, unit: 'SECONDS') { // change to a convenient timeout for you
		    userInput = input(
		        id: 'Proceed1', message: 'Was this successful?', parameters: [
		        [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']
		        ])
                }
                archiveArtifacts "sources/dist/add2vals"
                timeout(time: 1) {
                    input message: 'Sudah selesai menggunakan add2vals App?'
                }
                sh "docker run --rm -v ${env.VOLUME} ${env.IMAGE} 'rm -rf build dist'"
            }
        }
    }
}
