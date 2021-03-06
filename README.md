Requires [docker](https://www.docker.com/) &
[docker-compose](https://docs.docker.com/compose/). Linux is definitely a plus!

# Development servers

* [Bugzilla](https://dev.dashboard.unee-t.com)
* [Meteor](https://dev.case.unee-t.com)
* db.dev.unee-t.com

# Production servers

* [Bugzilla](https://dashboard.unee-t.com)
* [Meteor](https://case.unee-t.com)
* db.prod.unee-t.com

# Run

	make up
	make down

You might need to do a `make ${up,down,up}` to make it all work due to the
setup phases of {bugzilla,db} being a bit difficult to co-ordinate with docker compose.

# Login info

If you are using the default primed sql/demo state username;password are:

	administrator@example.com;administrator
	anabelle@example.com;anabelle
	celeste@example.com;celeste
	jocelyn@example.com;jocelyn
	lawrence@example.com;lawrence
	leonel@example.com;leonel
	marina@example.com;marina
	marley@example.com;marley
	marvin@example.com;marvin
	michael@example.com;michael
	regina@example.com;regina
	sabrina@example.com;sabrina

Else if not sql/demo see [bugzilla_admin](bugzilla_admin)

# Debug

	docker exec -it docker_bugzilla_1 /bin/bash

The prefix `docker_` might be different on your system.

# Database explorer

http://localhost:8082/ see [environment](.env) for credentials

# JSON API

<https://bugzilla.readthedocs.io/en/latest/api/>

	curl http://localhost:8081/rest/bug/1 | jq

Or upon the dev server:

	curl -s https://dev.unee-t.com/rest/bug/1?api_key=I6zRu7bPak687rcIBCkNbFKblfRXPn2X3xgEFz99 | jq

# Build

You shouldn't need to do this since normally we should use out gitlab hosted Bugzilla image.

	make build

# Environment

Secrets are managed in [AWS's parameter
store](https://ap-southeast-1.console.aws.amazon.com/ec2/v2/home?region=ap-southeast-1#Parameters:sort=Name).

Assuming the profiles `lmb-dev` is setup for development AWS account
8126-4485-3088 and `lmb-prod` for 192458993663, the `aws-env` and
`aws-env.prod` will be sourced and [aws-cli will fetch the secrets](https://github.com/aws/aws-cli/issues/2950).

* MONGO_PASSWORD
* MYSQL_PASSWORD
* SES_SMTP_PASSWORD
* BUGZILLA_ADMIN_KEY

`SES*` is required for email notifications. [SES dashboard](https://us-west-2.console.aws.amazon.com/ses/home?region=us-west-2#dashboard:)

How to test if email is working:

	echo -e "Subject: Test Mail\r\n\r\nThis is a test mail" | msmtp --debug -t user@example.com

Video about testing email: https://s.natalian.org/2017-10-27/uneetmail.mp4

# State snapshots

Save:

	mysqldump -h 127.0.0.1 -P 3306 -u root --password=uniti bugzilla > demo.sql

Restore:

	mysql -h 127.0.0.1 -P 3306 -u root --password=uniti bugzilla < fresh.sql

To restore on dev server:

	mysql -h db.dev.unee-t.com -P 3306 -u root --password=SECRET bugzilla < demo.sql

# How to check for mail when in test mode

	docker exec -it bugzilla_bugzilla_1 /bin/bash
	cat data/mailer.testfile

# AWS ECS setup

Cluster is named after branch name.

	ecs-cli configure -c $BRANCH -r ap-southeast-1 -p $PROFILE
	ecs-cli up --vpc vpc-efc2838b --subnets subnet-846169e0,subnet-cf7b46b9 --instance-type t2.medium --capability-iam --keypair hendry

For production:
	ecs-cli up --vpc vpc-b93070dd --subnets subnet-9ae9e1fe,subnet-fcffc28a --instance-type t2.medium --capability-iam --keypair hendry,franck

Refer to `ecs-cli compose service create -h` to create with a load balancer.

* [Development account](https://812644853088.signin.aws.amazon.com/console)
* [Production account](https://192458993663.signin.aws.amazon.com/console)
