# beanstalk-notify

Know what is going on with your Elastic Beanstalk in Slack when deploying to EC2 Beanstalks.  This EB setup was done with Python/Django in mind, but it will expand at some point.  

_**CAUTION TO THOSE WHO ENTER: This is fully a work in progress and is not polished. In fact the current slack notification works well, but the formatting leaves empty variables and look weird.  Like:
![See]
(http://i.imgur.com/UaZPFM4.png) So missing is the actual app version, who deplopyed (user, Jenkins, etc) and the environment of of the deploy.  Kinda importantt, and will be addressed ASAP.**_


____

Built off of the work done by Springest per their [blog][post] with the [repo here](https://github.com/Springest/elastic-beanstalk-deploy-notifications).

[post]: http://devblog.springest.com/deploy-notifications-to-newrelic-appsignal-and-slack-with-elastic-beanstalk/

This was created anew as the Springest code has some non-obious TLC needs as well seemed to head in the Docker direction for EBS.  

___

# Setup

The Slack integration uses Webhooks into Slack which allowed for posting to rooms.  Pretty easy, so head over to [Slack](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) and install that into your team, and get your webhook URL.  It'll be something like `https://hooks.slack.com/services/A0A0A0A0/B1B1B1B1/XYXYXYXYXYXYXYXYXYXYXYXY` and we'll use that later to add an Environment Property to the application in the Beanstalk console (or EB CLI).

This flow is based on using S3 as the store for any EBS setup scripts needed.  We are using the Django-Makeconf project by Ethan McCreadie to create modular EB configs based on per-project needs.  Makeconf uses .tmpl files which are then processed into the .ebextensions dir by running `./manage.py makeconf`.  The project as docs, [read them here](https://github.com/ethanmcc/django-makeconf).

You will need the following settings available to the EB config in the template:

FYI don't use a bucket namespaced with dots for full boto compatibility, [see Github issue here](https://github.com/boto/boto/issues/2836)

* `AWS_CONFIG_BUCKET`  
* `AWS_SLACK_SCRIPT`

Example:

`AWS_CONFIG_BUCKET = 'org-aws-configutations'`
`AWS_SLACK_SCRIPT = '/ebs/slack_deploy.sh'`

In addition Beanstalk must be able to access private repos, so the IAM Role is included in the setup: `roleName: {{ settings.EB_ROLENAME }}`

Again, replace the python settings notion with whatever works.

Currently there are a three inputs to the slack shell script that are added directly to the Beanstalk environment (yeah, that's change to be more dynamic) either via the console (you can figure that out) or via the EB CLI. 

You need:

* `APP_NAME`
* `SLACK_WEBHOOK_URL`
* `SLACK_CHANNEL`

The process for the EB CLI is straight forward (assuming the applciation environment already exists).  If not then create the EB environment with `eb create` first.  To setup the environment properties you simply run:
`eb setenv APP_NAME=<YOUR-APP-NAME> SLACK_CHANNEL=<WHATEVER_CHANNEL> SLACK_WEBHOOK_URL=<YOUR WEB HOOK URL>`

EB is ready to go. So, using the django-makeconf setup, you run `./manage.py makeconf --settings <path to settings file>` and let that create the .ebextensions dir and config files.  

From there you can `eb deploy` and watch your logs for any S3 issues should there be any IAM permissions that need tweaking so the EB IAM Role can get to your S3 bucket. 

No errors, no worries and your deploy will be posted into the target Slack channel


# To Do

* Fix formatting in current output 
* Clean up slack script and remove assumptions
   * Args should handle inputs dynamically (deploying on your local versus Jenkins for instance)
* EC2 Beanstalk environment name is broken in current version as it's relying on Docker
* Add more downstream notifications (Hipchat, New Relic, email, sms, etc)
* More output formatting options for sure
* Template file -> less python more...something that's across platforms
