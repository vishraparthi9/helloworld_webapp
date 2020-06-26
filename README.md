# helloworld_webapp

## How did I generate this project

```
mvn archetype:generate -DgroupId=com.vishraparthi.webapp -DartifactId=helloworld -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```

## Github webhook

[Receive Github webhooks on Jenkins without public IP](https://webhookrelay.com/blog/2017/11/23/github-jenkins-guide/)

Install Jenkins in Mac using following commands:

 * Install the latest Weekly version: brew install jenkins
 * Start the Jenkins service: brew services start jenkins
 * Update the Jenkins version: brew upgrade jenkins

More info in [here](https://www.jenkins.io/download/weekly/macos/)

Install Webhook relay:

 * Install Relay in Mac: brew install webhookrelay/tap/relay
 * Create account [here](https://my.webhookrelay.com/register) or login via
 github credentials
 * Once registered, generate a token key and secret as username and password for
 CLI authentication
 * Use it: `relay login -k token-key-here -s token-secret-here`
 * Start forwarding webhooks to jenkins: `relay forward --bucket github-jenkins http://localhost:8080/github-webhook/`