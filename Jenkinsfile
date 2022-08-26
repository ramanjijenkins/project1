pipeline {
  agent any

  environment {
    
    //WORKSPACE           = '.'
    //DBRKS_BEARER_TOKEN  = "xyz"
    DBTOKEN             = "databricks-token"
    CLUSTERID           = "AWS-Glue Catalogue Cluster"
    DBURL               = "https://dbc-db420c65-4456.cloud.databricks.com"

    TESTRESULTPATH  ="./teste_results"
    LIBRARYPATH     = "./Libraries"
    OUTFILEPATH     = "./Validation/Output"
    NOTEBOOKPATH    = "./Notebooks"
    WORKSPACEPATH   = "/Shared"
    DBFSPATH        = "dbfs:/FileStore/"
    BUILDPATH       = "${WORKSPACE}/Builds/${env.JOB_NAME}-${env.BUILD_NUMBER}"
    SCRIPTPATH      = "./Scripts"
    projectName = "${WORKSPACE}"  //var/lib/jenkins/workspace/Demopipeline/
    projectKey = "key"
      }

  stages {
    stage('Install Miniconda') {
        steps {

            sh '''#!/usr/bin/env bash
            echo "Inicianddo os trabalhos"  
            wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -nv -O miniconda.sh

            rm -r $WORKSPACE/miniconda
            bash miniconda.sh -b -p $WORKSPACE/miniconda
            
            export PATH="$WORKSPACE/miniconda/bin:$PATH"
            echo $PATH

            conda config --set always_yes yes --set changeps1 no
            conda update -q conda
            conda create --name mlops2
            
            echo ${BUILDPATH}

            '''
        }

    }

    stage('Install Requirements') {
        steps {
            sh '''#!/usr/bin/env bash
            echo "Installing Requirements"  
            source $WORKSPACE/miniconda/etc/profile.d/conda.sh
            conda activate mlops2

            export PATH="$HOME/.local/bin:$PATH"
            echo $PATH
            
            pip install --user databricks-cli
            pip install -U databricks-connect
            pip install pytest pyspark==3.3.0
            databricks --version

           '''
        }

    }
    
    stage('Setup') {
		steps{
		  withCredentials([string(credentialsId: DBTOKEN, variable: 'TOKEN')]) {
			sh """#!/bin/bash
                # Configure Databricks CLI for deployment
				        echo "${DBURL}
				        $TOKEN" | databricks configure --token

				        # Configure Databricks Connect for testing
				        echo "${DBURL}
				        $TOKEN
				        ${CLUSTERID}
				        0
				        15001" | databricks-connect configure
				        echo "list the workspace and below are the notebooks:"
				        databricks workspace ls /Users/emmanuel.mua@ibm.com
			   """
		  }
		}
	}
    
   

   
    stage('Unit Tests') {
      steps {

        script {
            try {
              sh """#!/bin/bash
                source $WORKSPACE/miniconda/etc/profile.d/conda.sh
                conda activate mlops2
		pip3 install nbmake pytest-xdist 
                # Python tests for libs
                python -m pytest --junit-xml=${TESTRESULTPATH}/TEST-libout.xml ${LIBRARYPATH}/python/dbxdemo/test*.py || true
		python3 -m pytest --nbmake -n=auto ${LIBRARYPATH}/python/dbxdemo/test*.ipynb || true
                """
          } catch(err) {
            step([$class: 'JUnitResultArchiver', testResults: '--junit-xml=${TESTRESULTPATH}/TEST-*.xml'])
            if (currentBuild.result == 'UNSTABLE')
              currentBuild.result = 'FAILURE'
            throw err
          }
        }
      }
    }

stage('build && SonarQube analysis') {
          steps {
            //def scannerhome = tool name: 'SonarQubeScanner'

            withEnv(["PATH=/usr/bin:/usr/local/jdk-11.0.2/bin:/opt/sonarqube/sonar-scanner/bin/"]) {
           withSonarQubeEnv('sonar') {
                     sh "/opt/sonar-scanner/bin/sonar-scanner -Dsonar.projectKey=sonar-project -Dsonar.projectVersion=0.0.2 -Dsonar.sources=${projectName} -Dsonar.host.url=http://107.20.71.233:9001 -Dsonar.login=ab9d8f9c15baff5428b9bf18b0ec198a5b35c6bb -Dsonar.python.coverage.reportPaths=coverage.xml -Dsonar.sonar.inclusions=**/*.ipynb -Dsonar.exclusions=**/*.ini,**/*.py,**./*.sh"
                }
              }
        }
        }

  } 
}
