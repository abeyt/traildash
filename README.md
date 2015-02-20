# Traildash: AWS CloudTrail Dashboard
Traildash is a powerful dashboard for AWS CloudTrail logs, shipped in an easy-to-use docker container.

![CloudTrail Dashboard](/readme_images/traildash_screenshot.png)

Configure Traildash with a few environment variables and you're off to the races.

## Quickstart
1. [Setup AWS services to support CloudTrail](#setup-cloudtrail-in-aws)
2. Fill in the "XXX" blanks and run with docker: 
```
docker run -i -d -p 7000:7000 \
	-e "AWS_ACCESS_KEY_ID=XXX" \
	-e "AWS_SECRET_ACCESS_KEY=XXX" \
	-e "AWS_SQS_URL=https://XXX" \
	-e "DEBUG=1" \
	-v /home/traildash:/var/lib/elasticsearch/ \
	appliedtrust/traildash
```
3. Open http://localhost:7000/ in your browser

#### Required Environment Variables:
	AWS_SQS_URL				AWS SQS URL.
	AWS_ACCESS_KEY_ID		AWS Key ID.
	AWS_SECRET_ACCESS_KEY	AWS Secret Key.

#### Optional Environment Variables:
	AWS_REGION		AWS Region (SQS and S3 regions must match.  default: us-east-1).
	ES_URL			ElasticSearch URL (default: http://localhost:9200).
	WEB_LISTEN		Listen IP and port for web interface (default: 0.0.0.0:7000).
	SQS_PERSIST		Set to prevent deleting of finished SQS messages - for debugging.
	DEBUG			Enable debugging output.

## Using traildash outside Docker
Download Kibana 3.x and uncompress it to "kibana" in the same directory you run `traildash` from.

#### Usage:
	traildash
	traildash --version

#### Example Environment Variables
```
export AWS_ACCESS_KEY_ID=AKIXXX
export AWS_SQS_URL=XXX
export AWS_SECRET_ACCESS_KEY=XXX
export AWS_REGION=us-east-1
export ES_URL=http://localhost:9200
export WEB_LISTEN=localhost:7000
export DEBUG=1
export SQS_PERSIST=1
```

## How it works 
1. AWS CloudTrail creates a new log file, stores it in S3, and notifies an SNS topic.
1. The SNS topic notifes a dedicated SQS queue about the new log file in S3.
1. Traildash polls the SQS queue and downloads new log files from S3.
1. Traildash loads the new log files into a local ElasticSearch instace.
1. Kibana provides beautiful dashboards to view the logs stored in ElasticSearch.
1. Traildash protects access to ElasticSearch, ensuring logs are read-only.

## Building
```
make linux
make kibana
make docker
```

## Setup CloudTrail in AWS
1. In your primary region, turn on CloudTrail: ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_01.png)
1. Tell CloudTrail to create a new S3 bucket and SNS topic: ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_02.png)
1. Switch to SNS in your AWS console to view your SNS topic and copy its ARN to your clipboard: ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_03.png)
1. Switch to SQS in your AWS console and create a new SQS queue - okay to stick with default options: ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_04.png) ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_05.png)
1. Select your SQS queue, click the "permissions" tab, then click "Add a Permission": ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_06.png)
1. Click "Everybody", and click the "SendMessage" action: ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_07.png)
1. Change the Key to "aws:SourceArn", paste in your SNS topic ARN from your clipboard, then click "Add Condition": ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_08.png)
1. Click "Add Permission": ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_09.png)
1. Copy your SQS queue's ARN to your clipboard: ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_10.png)
1. Switch back to your SNS topic, and click "Add Subscription": ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_12.png)
1. Paste in your SQS queue's URL from your clipboard:  ![CloudTrail setup](/readme_images/AWS_CloudTrail_Setup_13.png)
1. To add other regions:
  1. Configure all regions to use the same S3 bucket.
  1. Configure your SQS queue to permit each region's SNS topic.
  1. Subscribe your central SQS queue to each region's SNS topic.
