pipeline {
  
agent none  

stages{      

stage('worker build'){        
when{            
changeset "**/worker/**"          
}        
agent{          
docker{            
image 'maven:3.6.1-jdk-8-slim'
args '-v $HOME/.m2:/root/.m2'          
}        
}        
steps{          
echo 'Compiling worker app..'          
dir('worker'){            
sh 'mvn compile'          
}        
}      
}      

stage('worker test'){        
when{          
changeset "**/worker/**"        
}        
agent{          
docker{            
image 'maven:3.6.1-jdk-8-slim'            
args '-v $HOME/.m2:/root/.m2'          
}        
}        
steps{          
echo 'Running Unit Tets on worker app..'          
dir('worker'){            
sh 'mvn clean test'           
}          
}      
}      

stage('worker package'){        
when{          
branch 'master'          
changeset "**/worker/**"        
}        
agent{          
docker{            
image 'maven:3.6.1-jdk-8-slim'            
args '-v $HOME/.m2:/root/.m2'          
}
}        
steps{          
echo 'Packaging worker app'          
dir('worker'){            
sh 'mvn package -DskipTests'            
archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true          
}        
}      
}      

stage('worker-docker-package'){          
agent any          
when{            
changeset "**/worker/**"            
branch 'master'          
}          
steps{            
echo 'Packaging worker app with docker'            
script{              
docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') 
{                  
def workerImage = docker.build("kernetix/worker:v${env.BUILD_ID}", "./worker")                  
workerImage.push()                  
workerImage.push("${env.BRANCH_NAME}")                  
workerImage.push("latest")              
}            
}          
}      
}

stage('result build'){
when {
changeset "**/result/**"
}
agent{
docker{
image 'node:8.16.0:alpine'
}
}
steps{
echo 'Compiling result app'
dir('result'){
sh 'npm install'
}
}
}

stage('result test'){
when {
changeset "**/result/**"
}
agent{
docker{
image 'node:8.16.0:alpine'
}
}
steps{
echo 'Running Unit Tests on worker app'
dir ('result'){
sh 'npm test'
}
}
}

stage('result-docker-package'){          
agent any          
when{            
changeset "**/result/**"            
branch 'master'          
}          
steps{            
echo 'Packaging result app with docker'            
script{              
docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') 
{                  
def workerImage = docker.build("kernetix/result:v${env.BUILD_ID}", "./result")                  
workerImage.push()                  
workerImage.push("${env.BRANCH_NAME}")                  
workerImage.push("latest")              
}            
}          
}      
}

stage('vote build'){
when {
changeset "**/vote/**"
}
agent{
docker{
image 'python:2.7.16-slim'
args '--user root'
}
}
steps{
echo 'Compiling vote app'
dir('vote'){
sh 'pip install -i requirements.txt'
}
}
}

stage('vote test'){
when {
changeset "**/vote/**"
}
agent{
docker{
image 'python:2.7.16-slim'
args '--user root'
}
}
steps{
echo 'Running Unit Tests on vote app'
dir ('vote'){
sh 'nosetests -v'
}
}
}

stage('vote-docker-package'){          
agent any          
when{            
changeset "**/vote/**"            
branch 'master'          
}          
steps{            
echo 'Packaging vote app with docker'            
script{              
docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') 
{                  
def workerImage = docker.build("kernetix/vote:v${env.BUILD_ID}", "./vote")                  
workerImage.push()                  
workerImage.push("${env.BRANCH_NAME}")                  
workerImage.push("latest")              
}            
}          
}      
}
}  


post{
 always{
 echo 'Building monopipeline for instavote is completed..'
 }
 failure{
 slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
 }
 success{
 slackSend (channel: "instavote-cd", message: "Build Succeeded and OK - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
 }
 }
}
