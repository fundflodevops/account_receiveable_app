// app/Jenkinsfile

// Define parameters for the pipeline
def parameters = [
    string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Target environment (e.g., dev, staging, production)'),
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Whether to run tests'),
    choice(name: 'DEPLOY_TYPE', choices: ['full', 'incremental'], description: 'Type of deployment')
]

def runPipeline() {
    node {
        // Access the parameter values
        def environment = params.ENVIRONMENT
        def runTests = params.RUN_TESTS
        def deployType = params.DEPLOY_TYPE

        stage('Build') {
            echo "Building for environment: ${environment}..."
        }
        
        stage('Test') {
            if (runTests) {
                echo 'Running tests...'
            } else {
                echo 'Skipping tests...'
            }
        }
        
        stage('Deploy') {
            echo "Deploying using ${deployType} deployment strategy to ${environment}..."
        }
    }
}

// This will be called by the main Jenkins pipeline
return this
