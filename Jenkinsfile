pipeline{
	agent any
   	triggers {
       		pollSCM('*/5 * * * *')
  	}
	stages{
	      stage("Deliver to Docker Hub"){
		steps {
			sh"docker build . -t dpankov91/calculator"
			withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerHub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
			{
		   	sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
			}
			sh "docker push dpankov91/calculator"
		      }
		}
	      stage("Selenium grid setup") {
		steps{
			sh "docker network create SE"
			sh "docker run -d --rm -p 4444:4444 --net=SE --name selenium-hub selenium/hub"
			sh "docker run -d --rm --net=SE -e HUB_HOST=selenium-hub --name selenium-node-firefox1234 selenium/node-firefox" 
			sh "docker run -d --rm --net=SE -e HUB_HOST=selenium-hub --name selenium-node-chrome1234 selenium/node-chrome"
			sh "docker run -d --rm --net=SE --name app-test-container dpankov91/calculator"
		     }
	      }
	      stage("Execute system tests") {
		steps{
			sh "selenium-side-runner --server http://localhost:4444/wd/hub -c 'browserName=firefox' --base-url http://app-test-container test/system/CalculatorTest.side" 
			sh "selenium-side-runner --server http://localhost:4444/wd/hub -c 'browserName=chrome' --base-url http://app-test-container test/system/CalculatorTest.side"
		     }
	      }
	}
	post {
		cleanup {
		   echo "Cleaning the Docker environment"
		   sh script:"docker stop selenium-hub", returnStatus:true
		   sh script:"docker stop selenium-node-firefox", returnStatus:true
		   sh script:"docker stop selenium-node-chrome", returnStatus:true
		   sh script:"docker stop app-test-container", returnStatus:true
		   sh script:"docker network remove SE", returnStatus:true
			}
	 }
}
 
	      