// app/Jenkinsfile

// Define parameters for the pipeline

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
