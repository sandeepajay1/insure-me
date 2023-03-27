node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    
    stage('prepare environment'){
        echo 'initialize the variables'
        mavenHome = tool name: 'MAVEN_HOME' , type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'DOCKER' , type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName="3.0"
    }
    stage('code checkout'){
        try{
        echo 'code checkout'
        git 'https://github.com/sandeepajay1/insure-me.git'
        }
        catch(Exception e){
            echo 'exception occur in stage code checkout'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
            The jenkins job ${JOB_NAME} is failed. please look into the ${BUILD_NUMBER}''', subject: 'job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'sandeep.ajayakumar@gmail.com'
        }
    }
    stage('Build the application'){
        echo ' clean and compile and test package'
        //sh 'mvn clean package'
        sh "${mavenCMD} clean package"
    }
    
    stage('publish test reports'){
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/Staragile_Insure_me_project/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    }
    stage('Build the DockerImage of the application'){
        try{
            echo 'Creating the docker image'
            sh "${dockerCMD} build -t sandeepajay1/insure_me:${tagName} ."
            }
            catch(Exception e){
                echo 'Exception Occur'
                currentBuild.result = "failure"
                emailext body: '''Dear All,
                The jenkins job ${JOB_NAME} is failed. please look into the ${BUILD_NUMBER}''', subject: 'job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'sandeep.ajayakumar@gmail.com'
                
            }
    }
    stage('push the docker image to dockerhub'){
        echo 'pushing the docker image'
        withCredentials([string(credentialsId: 'DockerPassword', variable: 'DockerPassword')]) {
        sh "${dockerCMD} login -u sandeepajay1 -p ${DockerPassword}"
        sh "${dockerCMD} push sandeepajay1/insure_me:${tagName}"
        }
    }
    stage('deploy the application'){
        ansiblePlaybook become: true, credentialsId: 'ansiblekey', disableHostKeyChecking: true, installation: 'ANSIBLE', inventory: './inventory', playbook: 'ansible-playbook.yml'
    }
}
