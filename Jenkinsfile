pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '1'))
        disableConcurrentBuilds()
    }
    
    environment {
        REGTEST_JAR = 'jarfiles/RegTestRunner-8.10.5.jar'
    }
    
    triggers {
        pollSCM('H/5 * * * *')  // Poll GitHub every 5 minutes
    }
    
    parameters {
        choice(name: 'XUMLC', choices: ['jarfiles/xumlc-7.20.0.jar'], description: 'Location of the xUML Compiler')
        choice(name: 'REGTEST', choices: ['jarfiles/RegTestRunner-8.10.5.jar'], description: 'Location of the Regression Test Runner')
        string(name: 'BRIDGE_HOST', defaultValue: 'ec2-52-74-183-0.ap-southeast-1.compute.amazonaws.com', description: 'Bridge host address')
        string(name: 'BRIDGE_USER', defaultValue: 'jprocero', description: 'Bridge username')
        password(name: 'BRIDGE_PASSWORD', defaultValue: 'jprocero', description: 'Bridge password')
        string(name: 'BRIDGE_PORT', defaultValue: '11165', description: 'Bridge port')
        string(name: 'CONTROL_PORT', defaultValue: '21176', description: 'Control port')
    }


     
    stages {
        stage('Build') {
            steps {
                dir('.') {
                    bat """
                        java -jar ${params.XUMLC} -uml uml/BuilderUML.xml
                        if errorlevel 1 exit /b 1
                        echo Build completed successfully
                        dir repository\\BuilderUML\\*.rep
                    """
                    archiveArtifacts artifacts: 'repository/BuilderUML/*.rep'
                }
            }
        }
         stage('Deploy') {
            steps {
                dir('.') {
                    bat """
                        echo Checking for repository files...
                       
                        if not exist repository\\BuilderUML\\regtestlatest.rep (
                            echo ERROR: regtestlatest.rep not found!
                            exit /b 1
                        )
                         
                        echo All repository files found, starting deployment...
                        npx e2e-bridge-cli deploy repository/BuilderUML/regtestlatest.rep -h ${params.BRIDGE_HOST} -u ${params.BRIDGE_USER} -P ${params.BRIDGE_PASSWORD} -o overwrite
                        
                        echo Starting the deployed service...
                        npx e2e-bridge-cli start regtestlatest -h ${params.BRIDGE_HOST} -u ${params.BRIDGE_USER} -P ${params.BRIDGE_PASSWORD}
                        
                    """
                }
            }
        }
    }
}