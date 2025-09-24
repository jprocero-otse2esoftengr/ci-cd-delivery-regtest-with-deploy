pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '1'))
        disableConcurrentBuilds()
    }
    
    environment {
        REGTEST_JAR = 'jarfiles/RegTestRunner-8.10.5.jar'
        XUMLC = 'jarfiles/xumlc-7.20.0.jar'
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
                        java -jar ${XUMLC} -uml uml/BuilderUML.xml
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
                        echo Checking available commands...
                        where e2ebridge
                        where npx
                        echo Using native e2ebridge command (found in PATH)...
                        echo Force deploying service to ensure it's always updated...
                        e2ebridge deploy repository/BuilderUML/regtestlatest.rep -h ${BRIDGE_HOST} -u ${BRIDGE_USER} -P ${BRIDGE_PASSWORD} -o overwrite
                        
                        echo Stopping any existing service first...
                        e2ebridge stop regtestlatest -h ${BRIDGE_HOST} -u ${BRIDGE_USER} -P ${BRIDGE_PASSWORD} || echo No existing service to stop
                        
                        echo Removing any existing service to ensure clean deployment...
                        e2ebridge remove regtestlatest -h ${BRIDGE_HOST} -u ${BRIDGE_USER} -P ${BRIDGE_PASSWORD} || echo No existing service to remove
                        
                        echo Starting the deployed service...
                        e2ebridge start regtestlatest -h ${BRIDGE_HOST} -u ${BRIDGE_USER} -P ${BRIDGE_PASSWORD}
                        if errorlevel 1 (
                            echo ERROR: Failed to start service regtestlatest
                            exit /b 1
                        ) else (
                            echo Service regtestlatest started successfully
                        )
                        echo Service start command completed
                        
                        echo Waiting 5 seconds for service to initialize...
                        timeout /t 5 /nobreak
                        
                        echo Setting service preferences for automatic startup...
                        e2ebridge preferences regtestlatest --pref.automaticStartup=true -h ${BRIDGE_HOST} -u ${BRIDGE_USER} -P ${BRIDGE_PASSWORD}
                        
                        echo Restarting service to ensure all changes are applied...
                        e2ebridge restart regtestlatest -h ${BRIDGE_HOST} -u ${BRIDGE_USER} -P ${BRIDGE_PASSWORD}
                        
                        echo Checking service status...
                        e2ebridge status regtestlatest -h ${BRIDGE_HOST} -u ${BRIDGE_USER} -P ${BRIDGE_PASSWORD}
                        echo Service status check completed
                        
                    """
                }
            }
        }
    }
}