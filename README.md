# aws-ec2-docker-swarm-instructions

* start new ec2 instance (i recommend this: Amazon Linux 2 AMI 2.0.20190823.1 x86_64 HVM gp2)
    * When launching the instance, uncheck “delete on termination” at 4. Add Storage to preserve data in case of instance restarts
    * The rest leave default
* log in to the ec2
    * install git
    ```
    sudo yum install git -y
    ```
    * install docker
    ```
    sudo yum update -y
    sudo amazon-linux-extras install docker
    sudo service docker start
    sudo usermod -a -G docker ec2-user
    ```
    * install ssm agent (for travis to be able to execute commands)
    ```
    sudo yum install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
    ```
* log out and log in again (to be able to execute docker commands without sudo)
    * clone your repo: 
    ```
    sudo git clone https://github.com/billiamdan/warmeup.git
    ```
    * pull your docker image
    ```
    docker pull billiamdan/warmeup:latest # check exact image name in your docker hub
    ```
    * start docker swarm cluster
    ```
    docker swarm init --advertise-addr <private ip address of the ec2> # can be found in the Description section of the ec2
    ```
    * deploy your stack of services using docker-compose.depl.yml
    ```
    docker stack deploy shadowMarathonStackk -c docker-compose.depl.yml
    ```
    * check that the stack is running alright
    ```
    watch docker stack ps shadowMarathonStackk 
    ```
* create User in AWS IAM to be able to execute commands with travis
    * create a new policy
        * go to IAM => policies
        * create new policy (see policy below)
        * replace us-east-2:650732200008 with region you ec2 instance is located at and your account id (can be seen in the Owner section of ec2 description)
    * create group 
        * go to IAM => groups
        * create new group
            * when creating the group attach the policy created above to the group
    * create a user 
        * go to IAM => users
        * add new user
            * check checkbox Programmatic access
            * in permissions step add the user to the group created above
            * skip tags for now
            * at the next step you’ll see Access key ID and Secret access key
                * download them as those are needed for Travis to access your ec2
* set up travis env variables
    * INSTANCE_ID => get it in the Description section of your ec2
    * AWS_ACCESS_KEY => the Access key ID from above
    * AWS_SECRET_KEY => Secret access key from above



```
POLICY:
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ssm:SendCommand",
                "ssm:DescribeInstanceInformation"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:ec2:us-east-2:650732200008:instance/*",
                "arn:aws:ssm:us-east-2::document/AWS-RunShellScript"
            ]
        }
    ]
}
```
