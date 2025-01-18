pipeline {
    agent any
    
    environment {
        GIT_REPO = 'https://github.com/teja-tgh/cicd-demo-test-20.git'
        APPROVAL_EMAIL = 'teja.dannina@gmail.com'
    } 
    
    stages {
        stage('Build Application') { 
            steps { 
                bat 'mvn clean install' 
            } 
        }
        stage('Publish to exchange') { 
            steps { 
            	script {
                	if(env.GIT_BRANCH.toLowerCase() == 'origin/dev') {
    					bat 'mvn clean deploy -DskipTests'
    				}
		
					else {
						echo 'Application already deployed in dev process' 
					}
				}
            } 
	}


     stage('Approval') {
            steps {
                script {
                    if(env.GIT_BRANCH.toLowerCase() == 'origin/prod') {
                        sendApprovalEmail()
                        def approvalResponse = input(
                            id: 'userInput',
                            message: 'Please approve the deployment to dev environment',
                            parameters: [
                                [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Approve Deployment?', name: 'Approve']
                            ],
                            timeout: 5 * 60, // 5 minutes timeout
                            timeoutMessage: 'Approval timed out. Deployment not approved.'
                        )
                        if (!approvalResponse) {
                            error 'Deployment not approved. Aborting.'
                        }
                    } else{
                    echo "Approval Not needed"
                    }
                }
            }
        }
        stage('Build and Deploy') {
            steps {
                script {
                    if(env.GIT_BRANCH.toLowerCase() == 'origin/dev') {
                    deployToAnypoint('Sandbox','cicd-demo-test-20-dev')
                    }
                    else if(env.GIT_BRANCH.toLowerCase() == 'origin/qa') {
                    deployToAnypoint('Sandbox','cicd-demo-test-20-qa')
                    }
                    else if(env.GIT_BRANCH.toLowerCase() == 'origin/prod') {
                    deployToAnypoint('Sandbox','cicd-demo-test-20-prod')
                    }
                    else {
                    echo "Branch not configured"
                    echo env.GIT_BRANCH
                    
                    }
                }
            }
        }
    }
	post {
	    success {
	        mail to: 'teja.dannina@gmail.com', 
	        subject: "Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
	        mimeType: 'text/html',
	        body: """
	             <html>
	                 <body>
	                     <h2 style="color: green;">Jenkins Build Successful!</h2>
	                     <p>Good news! The Jenkins build <strong>'${env.JOB_NAME}'</strong> #<strong>${env.BUILD_NUMBER}</strong> was successful.</p>
	                     <hr />
	                     <p><strong>Details:</strong></p>
	                     <ul>
	                         <li><strong>Job Name:</strong> ${env.JOB_NAME}</li>
	                         <li><strong>Build Number:</strong> ${env.BUILD_NUMBER}</li>
	                         <li><strong>Build Time:</strong> ${new Date()}</li>
	                         <li><strong>Triggered By:</strong> ${currentBuild.getBuildCauses()[0]?.userId ?: 'Automated Trigger'}</li>
	                         <li><strong>Branch:</strong> ${env.GIT_BRANCH ?: 'N/A'}</li>
	                         <li><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
	                     </ul>
	                     <p style="color: green;">Keep up the great work!</p>
	                 </body>
	             </html>
	             """
	    }
	    failure {
	        mail to: 'teja.dannina@gmail.com',
	             subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
	             mimeType: 'text/html',
	             body: """
	             <html>
	                 <body>
	                     <h2 style="color: red;">Jenkins Build Failed</h2>
	                     <p>Unfortunately, the Jenkins build <strong>'${env.JOB_NAME}'</strong> #<strong>${env.BUILD_NUMBER}</strong> failed.</p>
	                     <hr />
	                     <p><strong>Details:</strong></p>
	                     <ul>
	                         <li><strong>Job Name:</strong> ${env.JOB_NAME}</li>
	                         <li><strong>Build Number:</strong> ${env.BUILD_NUMBER}</li>
	                         <li><strong>Build Time:</strong> ${new Date()}</li>
	                         <li><strong>Triggered By:</strong> ${currentBuild.getBuildCauses()[0]?.userId ?: 'Automated Trigger'}</li>
	                         <li><strong>Branch:</strong> ${env.GIT_BRANCH ?: 'N/A'}</li>
	                         <li><strong>Build URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></li>
	                     </ul>
	                     <p style="color: red;">Please review the logs and take necessary actions.</p>
	                 </body>
	             </html>
	             """
	    }
	}
	
}

def deployToAnypoint(environmentName,appName) {
    echo environmentName
    echo 'Deploying mule project due to the latest code commit…' 
    echo 'Deploying to the configured environment….' 
    bat """
        mvn clean deploy -DmuleDeploy \
        -Dmule_env=${environmentName} \
        -Dmule_appName=${appName}
    """
    }
def sendApprovalEmail() {
    def subject = "Approval Required: Jenkins Pipeline Deployment to Production Environment"
    def body = """
        <html>
        <body>
        <p>Dear Approver,</p>
        <p>A deployment to the Production environment is pending your approval.</p>
        <p>Please approve or deny using the Jenkins input step in the build page.</p>
        <p>Branch: ${env.GIT_BRANCH}</p>
        <p>View the build at: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
        <p>Thank you.</p>
        </body>
        </html>
    """
    mail to: "${APPROVAL_EMAIL}", subject: subject, body: body, mimeType: 'text/html'
}