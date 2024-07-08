
# Replica for Dapen Telkom Mobile in AWS

This guide outlines the steps to configure and automate an AWS replica for the Dapen Telkom Mobile server. The setup ensures that if the primary server crashes, the replica in AWS will automatically start and send a notification email.

## Prerequisites

- An AWS account
- AWS CLI installed on the primary server
- Mail utilities installed on the primary server

## Step-by-Step Instructions

## 1. Install AWS CLI on the Primary Server

Update the package list and install AWS CLI:

```sh
sudo apt-get update
sudo apt-get install awscli
```

## 2. Configure AWS CLI
### Configure AWS CLI with your credentials:
```sh
aws configure
```
## Create a Script to Start the Replica
### Create a script named start_replica.sh to start the replica instance in AWS and send an email notification:

example 
start_replica.sh

```sh 
#!/bin/bash

# Instance ID of the AWS replica
INSTANCE_ID="i-1234567890abcdef0"
EMAIL="dapentelkom@example.com"

# Start the replica instance
aws ec2 start-instances --instance-ids $INSTANCE_ID

# Wait for some time to ensure the instance starts
sleep 60

# 60 second
# 120 second is 1 minute

# Check the instance status
INSTANCE_STATUS=$(aws ec2 describe-instance-status --instance-ids $INSTANCE_ID --query 'InstanceStatuses[*].InstanceState.Name' --output text)

# If the instance is running, send an email
if [ "$INSTANCE_STATUS" == "running" ]; then
    echo "AWS replica has been successfully started." | mail -s "AWS Replica Report" $EMAIL
else
    echo "Failed to start AWS replica. Status: $INSTANCE_STATUS" | mail -s "AWS Replica Report" $EMAIL
fi
```
### Make the script executable:
```sh 
chmod +x /path/to/start_replica.sh
```
## 4. Install Mail Utilities on the Primary Server
### Update the package list and install mail utilities:

```sh
sudo apt-get update
sudo apt-get install mailutils
```
## 5. Monitor the Primary Server and Start the Replica
### Create a script named monitor_and_start_replica.sh to monitor the primary server and start the replica if the primary server is down:
```sh

#!/bin/bash

# IP or hostname of the primary server
SERVER_IP="dapentelkom_server_ip"
LOG_FILE="/path/to/replica.log"

# Ping the primary server
ping -c 1 $SERVER_IP > /dev/null 2>&1
if [ $? -ne 0 ]; then
    # If the server is not responding, run the script to start the replica
    /path/to/start_replica.sh >> $LOG_FILE 2>&1
fi

```
### Make the script executetable :
```sh
chmod +x /path/to/monitor_and_start_replica.sh
```

## 6. Set Up Cron Job to Monitor the Primary Server
### Edit the crontab to run the monitoring script every minute:
```sh
crontab -e
```
### Add the following entry:
```sh
* * * * * /path/to/monitor_and_start_replica.sh
```

## 7. Set Up Nagios (Optional)
### If you are using Nagios for monitoring, add the following command and service definitions

### Command Definition

```sh
define command{
    command_name    start_replica
    command_line    /path/to/start_replica.sh
}
```

### Service Definition

```sh
define service{
    use                 generic-service
    host_name           ip_dapen_mobile
    service_description Check Server
    check_command       check-host-alive
    event_handler       start_replica
}

```

## Example running in exisiting server dapen telkom mobile 
```sh
#!/bin/bash

# IP or hostname from main server
SERVER_IP="dapentelkom_server_ip"

# Ping main server
ping -c 1 $SERVER_IP > /dev/null 2>&1
if [ $? -ne 0 ]; then
    # If the server is not responding, run the script to start the replica
    /path/to/start_replica.sh >> $LOG_FILE 2>&1
fi
```

Running, Test, and Check Mail


## 8. Test the Setup
Manually stop the primary server to simulate a failure.
Verify that the replica instance in AWS starts automatically.
Check your email for the notification.
Review the log file to ensure the script executed without errors.

## 9. Monitor Logs and Email Notifications
Ensure that your monitoring script logs are updated and emails are being received as expected:

Log file location: /path/to/replica.log
Email notifications should be sent to the configured email address.
By following these steps, you can ensure that the AWS replica for Dapen Telkom Mobile is automatically started in the event of a primary server failure, with a notification email sent to inform you of the status.

#### This flow, execution and implemetation is created by Hendra Darisman, Pikamedia Indonesia. For more information, contact info@pikamedia.id.
