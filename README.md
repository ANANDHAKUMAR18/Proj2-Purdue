The recommended git tool is: NONE
No credentials specified
Cloning the remote Git repository
Cloning repository https://github.com/ANANDHAKUMAR18/Proj2-Purdue.git
 > /usr/bin/git init /var/lib/jenkins/workspace/Purdue-Pipeline # timeout=10
ERROR: Error cloning remote repo 'origin'
>
>
>
> pipeline
{
	agent any
	stages
	{
		stage('Code Checkout')
		{
			steps
			{
				git 'https://github.com/jsachdev07/purdue-igp.git'
			}
		}
		
		stage('Code Compile')
		{
			steps
			{
				sh 'mvn compile'
			}
		}

		stage('Test')
		{
			steps
			{
				sh 'mvn test'
			}
		}

		stage('Build')
		{
			steps
			{
				sh 'mvn package'
			}
		}
   }
}
