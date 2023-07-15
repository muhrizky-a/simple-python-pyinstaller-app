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
    docker.image('cdrx/pyinstaller-linux:python2').inside() {
        try {
            stage('Deploy') {
                sh 'pyinstaller --onefile sources/add2vals.py'
            }
        } catch (e) {
            echo 'This will run only if failed'
            throw e
        } finally {
            echo 'This will always run'
        }
    }
}
