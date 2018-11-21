pipeline {
  agent any
  stages {
    stage('Paso 1: inicio') {
      steps {
        echo 'Entrando a pipeline'
      }
    }
    stage('Paso 2: rama de ejecución') {
      steps {
        echo 'Rama ' + env.BRANCH_NAME
      }
    }
    stage ("Paso 3: Rama de desarrollador") {
      // si no es la rama master entonces ejecuta la integración continua
      when { not { branch 'master' } }
      steps {
        echo "entrando a hacer el pull request y merge"
        withCredentials([sshUserPrivateKey(credentialsId: '20f8159b-d214-48c3-9f07-4ae2aa3af5a9', keyFileVariable: '', passphraseVariable: '', usernameVariable: '')]) {
          sh 'git remote set-url origin https://Madesoft:Madesoft2018*@github.com/Madesoft/project_saleor.git'
          sh 'git fetch origin'
          sh 'git checkout origin/master'
          sh 'git pull . origin/' + "${env.BRANCH_NAME}" + ' --allow-unrelated-histories'
          sh 'git merge origin/' + "${env.BRANCH_NAME}"
          sh 'git push origin HEAD:master'
          echo 'Proceso de integración continua terminado exitosamente'
        }
      }
    }
    stage ("Paso 3: Rama master") {
      // si es la rama master no se hace integración
      when { branch 'master'}
      steps {
        echo 'Sólo se ejecuta en ramas de desarrolladores'
      }
    }
    stage('Update environment') {
        steps {
            sh '''source /home/jenkins/development/environments/project_saleor_env/bin/activate
                  pip install -r requirements.txt
                  pip install -r requirements_dev.txt
                  export SECRET_KEY='saleor'
                  python manage.py migrate
                '''
        }
    }
    stage('Static code metrics') {
        steps {
            sh '''source /home/jenkins/development/environments/project_saleor_env/bin/activate
                  radon raw --json saleor/ > reports/raw_report.json
                  radon cc --json saleor/ > reports/cc_report.json
                  radon mi --json saleor/ > reports/mi_report.json
                '''
        }
    }
    /*stage('Python unit tests') {
        steps {
            echo 'python unit tests'
            //pytest --cov-report html:./reports/cov_html --cov-report xml:./reports/cov.xml --cov=saleor --junitxml=./reports/results.xml test_ma0.py tests/test_translation.py  tests/test_account.py tests/test_collection.py
            sh '''source /home/jenkins/development/environments/project_saleor_env/bin/activate
                  pytest --cov-report html:./reports/cov_html --cov-report xml:./reports/cov.xml --cov=saleor --junitxml=./reports/results.xml test_ma0.py
                  '''
        }
        post{
            always{
                step([$class: 'CoberturaPublisher',
                               autoUpdateHealth: false,
                               autoUpdateStability: false,
                               coberturaReportFile: 'reports/cov.xml',
                               failNoReports: false,
                               failUnhealthy: false,
                               failUnstable: false,
                               maxNumberOfBuilds: 10,
                               onlyStable: false,
                               sourceEncoding: 'ASCII',
                               zoomCoverageChart: false])
              // Archive unit tests for the future
              junit allowEmptyResults: true, testResults: 'reports/results.xml'
            }
        }
    }*/
    stage ('Paso 4: Despliegue') {
      steps {
        echo 'Comenzando etapa de despliegue'
        echo 'Preparando copia de seguridad'
        def remote = [:]
        remote.name = 'test'
        remote.host = 'test.domain.com'
        remote.user = 'root'
        remote.password = 'password'
        remote.allowAnyHosts = true
        sshCommand remote: remote, command: "cp -r /home/saleor-produccion/dist /home/temp_deploy/"
        echo 'Copia de seguridad terminada'
        echo 'Preparando archivos para despliegue'
        sshPublisher(publishers: [sshPublisherDesc(configName: 'Deploy', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'echo "despliegue de archivos terminado"', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '**/*')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        echo 'Fase de despliegue terminada'
      }
    }
  }
  post {
    always {
      echo 'Finalizando proceso'
    }
    success {
      echo 'Proceso ' + "${env.BUILD_NUMBER}" + ' en ' + "${env.JOB_NAME}" + ' ha terminado exitosamente'
      mail bcc: '', body: "<b>Integración en código exitosa</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: 'madesoft.2.2018@gmail.com', mimeType: 'text/html', replyTo: '', subject: "SUCCESS CI: Project name -> ${env.JOB_NAME}", to: "anderson.enriquez@correounivalle.edu.co";
    }
    failure {
      echo 'Proceso ' + "${env.BUILD_NUMBER}" + ' en ' + "${env.JOB_NAME}" + ' presentó errores'
      mail bcc: '', body: "<b>Se presentó error en la integración del código</b><br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: 'madesoft.2.2018@gmail.com', mimeType: 'text/html', replyTo: '', subject: "ERROR CI: Project name -> ${env.JOB_NAME}", to: "anderson.enriquez@correounivalle.edu.co";
    }
  }
}
