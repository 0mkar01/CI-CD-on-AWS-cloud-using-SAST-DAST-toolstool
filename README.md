# CI/CD on AWS Cloud using SAST tool 

### Pre-requisites :

* AWS Account
* Sonar Cloud Account
* AWS CLI, Git installed on Local

![](images/Project-6.png)

### Step 1 :  Setup AWS CodeCommit 

From AWS Console, and pick `ap-south-1` region and go to `CodeCommit` service. Create repository.
```sh
Name: vprofile-code-repo
```

Next we will create an IAM user with `CodeCommit` access from IAM console. We will create a policy for `CodeCommit` and allow full access only for `vprofile-code-repo`.

```sh
Name: vprofile-code-admin-repo-fullaccess
```

![](images/iam-codecommit-admin-user.png)

To be able connect our repo using SSH, we will follow steps given in CodeCommit.

First Create SSH key in local and add public key to IAM role Security credentials.
```sh
cd .ssh/ 
ssh-keygen -t rsa -b 2048 -C "aws-code-commit"
```
Provide name: `aws-code-commit`
``` 
cat aws-code-commit.pub
```

![](images/sshkey-generated-local.png)

We will also update configuration under `.ssh/config` and add our Host information. And change permissions with `chmod 600 config`
```sh
Host git-codecommit.ap-south-1.amazonaws.com
    User <SSH_Key_ID_from IAM_user>
    IdentityFile ~/.ssh/aws-code-commit
```

We can test our ssh connection to CodeCommit.
```sh
ssh git-codecommit.ap-south-1.amazonaws.com
```

![](images/codecommit-ssh-connection-successful.png)

Next we clone the repository to a location that we want in our local. I will use the Github repository for `vprofile-project` in my local, and turn this repository to CodeCommit repository. When I am in Github repo directory, I will run below commands.

```sh
git checkout master
git remote rm origin
git remote add origin ssh://git-codecommit.ap-south-1.amazonaws.com/v1/repos/vprofile-code-repo
cat .git/config
git push origin --all
git push --tags
```
- Our repo is ready on CodeCommit with all branches.

![](images/codecommit-repo-ready.png)

### Step 2 : Setup AWS CodeArtifact

- We will create CodeArtifact repository for Maven.
```sh
Name: vprofile-maven-repo
Public upstream Repo: maven-central-store
This AWS account
Domain name: visualpath
```
- Again we will follow connection instructions given in CodeArtifact for  `maven-central-repo`.

![](images/artifact-connection-steps.png)

- We will need to create an IAM user for CodeArtifact and configure aws cli with its credentials. We will give Programmatic access to this user to be able to use aws cli and download credentials file.
```sh
aws configure # provide iam user credentials
```
![](images/iam-cart-admin-user.png)

Then we run command get token as in the instructions.
```sh
export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain visualpath --domain-owner 778343065186 --region ap-south-1 --query authorizationToken --output text`
```

Update pom.xml and setting.xml file with correct urls as suggested in instruction and push these files to codeCommit.
```sh
git add .
git commit -m "message"
git push origin ci-aws
```
### Step 3 : Setup SonarCloud 

We need to have an account, from account avatar -> `My Account` -> `Security`. Generate token name as `vprofile-sonar-cloud`. Note the token.

![](images/sonar-token.png)

Next we create a project, `+` -> `Analyze Project` -> `create project manually`. Below details will be used in our Build.
```sh
Organization: CDAC
Project key: vprofile
Public
```

Our Sonar Cloud is ready!
![](images/sonar-cloud-ready.png)

### Step-4: Store Sonar variables in System Manager Parameter Store 

We will create paramters for below variables.
```sh
CODEARTIFACT_TOKEN	 SecureString	
HOST                 https://sonarcloud.io
ORGANIZATION         CDAC
PROJECT              vprofile
SONARTOKEN           SecureString
```
