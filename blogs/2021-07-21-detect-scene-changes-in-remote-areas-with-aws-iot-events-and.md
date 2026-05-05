---
title: "Detect scene changes in remote areas with AWS IoT Events and Amazon SageMaker"
url: "https://aws.amazon.com/blogs/iot/detect-scene-changes-in-remote-areas/"
date: "Wed, 21 Jul 2021 21:56:06 +0000"
author: "Mehdi Amrane"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-events/feed/"
---
<p>Organizations with large numbers of assets need to monitor their physical and operational health, in order to detect issues and act upon them. This post covers the use case of a fictitious industrial organization AcmeDrone that uses drone devices to inspect assets periodically such as infrastructure components like valves, oil/gas pipelines or power transmission lines, located in hard-to-access areas. These inspections consist of having drone devices capture scene images of the assets and validating them against a machine learning model that detects changes to the asset, such as physical damage or physical obstacles that may affect the proper asset’s operation.</p> 
<p>In this post, we provide an operational overview of the aforementioned asset inspection solution, and then describe how to set up the applicable AWS IoT services:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/iot-events/">AWS IoT Events</a> to manage the asset’s geolocation route</li> 
 <li><a href="https://aws.amazon.com/sagemaker/">Amazon SageMaker</a> to detect scene changes</li> 
 <li><a href="https://aws.amazon.com/s3/">Amazon S3</a> and <a href="https://aws.amazon.com/lambda/">AWS Lambda</a> function to upload asset images and inspect them</li> 
 <li><a href="https://aws.amazon.com/sns/">Amazon Simple Notification Service (SNS)</a> to receive notifications of scene changes</li> 
 <li><a href="https://aws.amazon.com/iot-core/">AWS IoT Core</a> for devices’ communication with AWS</li> 
</ul> 
<h2><strong>Solution overview</strong></h2> 
<p>Consider a scenario where a drone device must follow a specific route to inspect various assets within a large industrial facility.&nbsp;During the inspection, the drone device would stop at each location on the route, in order to capture scene images of the assets. Those images will be sent to AWS and used to detect scene changes. If significant changes are detected, the solution notifies the asset operators.</p> 
<p>The drone device uses AWS IoT Core to authenticate with AWS and uses the <a href="https://github.com/aws/aws-iot-device-sdk-python">AWS IoT Device SDK for Python</a>.</p> 
<p>This asset inspection solution involves the following&nbsp;sequence of data and message exchanges:</p> 
<ol> 
 <li>The drone device publishes a notification to AWS IoT Core over a MQTT topic, in order to start the inspection route.</li> 
 <li>The <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-rules.html">AWS IoT Core rules engine</a> retrieves the notification from the MQTT topic.</li> 
 <li>The AWS IoT Core rules engine sends the notification to AWS IoT Events.</li> 
 <li>AWS IoT Events has a detector model that monitors incoming IoT events (e.g. drone device notifications) to start the inspection route (e.g. move the device to the recording scene) by sending a request back to the AWS IoT Core MQTT topic.</li> 
 <li>The device reads the request from the MQTT topic.</li> 
 <li>The device physically moves to a recording area and captures the scene image of the asset.</li> 
 <li>The device uploads the scene image to an Amazon S3 bucket.</li> 
 <li>Upon the scene image upload in the bucket, an AWS Lambda function gets executed.</li> 
 <li>The AWS Lambda function requests Amazon SageMaker to detect scene changes in the image uploaded by validating it against a model defined and deployed in Amazon SageMaker.</li> 
 <li>If scene changes were detected, the AWS Lambda function sends a notification message to an Amazon Simple Notification Service (SNS) topic.</li> 
 <li>The end-users subscribed to the SNS topic receive a notification message to inform them of the asset scene changes.</li> 
</ol> 
<p><img alt="Solution diagram" class="aligncenter wp-image-5454 size-large" height="933" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/07/14/IOT_Blog_Gear2-16-10-11-1-1024x933.png" width="1024" /></p> 
<h2><strong>Pre-requisites</strong></h2> 
<p>To follow along and set up the asset inspection solution, be sure to have the following:</p> 
<ul> 
 <li>An AWS account.</li> 
 <li>A device or laptop/computer with an access to your AWS account, Python installed, and the <a href="https://github.com/aws/aws-iot-device-sdk-python">AWS IoT Device SDK for Python</a>.</li> 
</ul> 
<h2><strong>To set up AWS IoT Events to manage drone device route</strong></h2> 
<p>In AWS IoT Events, we create the following components to start the drone device’s route:</p> 
<ul> 
 <li>One input representing the notification, sent by the drone device to the AWS Cloud.</li> 
 <li>A detector model with two states (and two transitions). Upon receipt of a notification, the device state is changed to “Area1”. This state is used to trigger the drone to move to the recording area.</li> 
</ul> 
<p>The following sections explain how&nbsp;the above components are implemented and how to configure AWS IoT Core to forward the device notification to AWS IoT Events.</p> 
<h2><strong>To create an input in AWS IoT Events</strong></h2> 
<p>You can create an input in AWS IoT Events by following the guide to <a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/iotevents-detector-input.html">create an input</a>. In our example, we create an input with the following details:</p> 
<ul> 
 <li>An Input name set to “deviceNotifInput”.</li> 
 <li>An example JSON payload/event with the content below:</li> 
</ul> 
<pre><code class="lang-json">{
  "deviceID": "1",
  "notifType": "StartRoute",
  "dateTm": "05/26/2021  08:03:20.200"
}</code></pre> 
<h2><strong>To create and publish a detector model in AWS IoT Events</strong></h2> 
<p>In our example, we create a detector model with the following details:</p> 
<ul> 
 <li>Two states (Stopped and Area1).</li> 
 <li>Two transitions (StartTransition and StopTransition) in order to transition the device from one state to another.</li> 
 <li>Upon the reception of a notification from the device, the start transition (StartTransition) triggers the sending of a notification to the outbound AWS IoT Core’s MQTT topic (See Steps 1 to 4 in previous section for further details) and changes the device state from Stopped to Area1.</li> 
</ul> 
<p><img alt="" class="size-full wp-image-5445" height="482" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/07/14/iot_events_diagram.jpg" width="1409" /></p> 
<p>To create a detector model,&nbsp;you can download this <a href="https://github.com/aws-samples/detect-scene-changes-aws-iot-events/blob/main/detector_model/iot_detector_model.json">sample detector model file</a>&nbsp;and import it into AWS IoT Events by following these steps:</p> 
<ol> 
 <li>Create an IAM role for the Detector Model. For more information, see the documentation for <a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/iotevents-start.html#iotevents-permissions-console">setting up permissions for AWS IoT Events</a>.</li> 
 <li>Download the sample detector model file from the repository at this <a href="https://github.com/aws-samples/detect-scene-changes-aws-iot-events/blob/main/detector_model/iot_detector_model.json">location</a>.</li> 
 <li>Update the file with the Detector Model IAM role’s ARN (replace value “#detectorModelRoleArn#”with a value in the following format: arn:aws:iam::<strong>account</strong>:role/service-role/<strong>detectorRoleName</strong>).</li> 
 <li>Go to AWS Management Console and select AWS IoT Events.</li> 
 <li>Select<em> Create detector model</em> and select <em>Import detector model</em>.</li> 
 <li>Select <em>Import</em>, select the file from your local system and select Open. This will create your Detector Model.</li> 
 <li>Once the Detector Model is created, you can publish it by following the guide <a href="https://docs.aws.amazon.com/pt_br/iotevents/latest/developerguide/iotevents-detector-model.html">Create a Detector Model</a>. In our example, the detector generation model is set to “Create a detector for each key value” with a key set to the deviceID. The Detector evaluation model is set to “Batch evaluation” as shown in the below screenshot.</li> 
</ol> 
<p><img alt="Publish detector model" class="wp-image-5403 size-full" height="727" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/07/14/publish_detector_model.jpg" width="972" /></p> 
<h2><strong>To configure AWS IoT Core rules</strong></h2> 
<p>An AWS IoT Core rule needs to be configured to forward device data from AWS IoT Core (MQTT topic) to AWS IoT Events.</p> 
<ol> 
 <li>Go to the AWS Management Console and select AWS IoT Core.</li> 
 <li>Select <em>Act</em>, then <em>Rules</em>, and then select the <em>Create</em> button.</li> 
 <li>Set the Name for the rule and set the rule query statement to <em>SELECT * FROM ‘<strong>deviceInboundTopic</strong></em>’.</li> 
 <li>Select “<em>Add action</em>”.</li> 
 <li>Select <em>Send a message to an IoT Events Input</em> and then select <em>Configure action</em> as shown in the below screenshot.</li> 
 <li>Select the Input previously created.</li> 
 <li>Select <em>Create Role</em> and enter a role name.</li> 
 <li>Select <em>Add action</em>.</li> 
 <li>Select <em>Create rule</em>.</li> 
</ol> 
<p><img alt="Configure AWS IoT Core rule" class="aligncenter wp-image-5405 size-full" height="703" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/07/14/iot_rules.jpg" width="1141" /></p> 
<h2><strong>To set up Amazon SageMaker to detect scene changes</strong></h2> 
<p>Amazon SageMaker builds a model that evaluates if an asset (i.e. scene image of the asset) changed.</p> 
<p>To create an Amazon SageMaker model, you can download this <a href="https://github.com/aws-samples/detect-scene-changes-aws-iot-events/blob/main/sagemaker_model/iot-notebook.ipynb">notebook</a> and follow the steps:</p> 
<ol> 
 <li>Create an Amazon SageMaker Notebook Instance by following the steps here: <a href="https://docs.aws.amazon.com/sagemaker/latest/dg/gs-setup-working-env.html">Create an Amazon SageMaker Notebook Instance</a>.</li> 
 <li>Once done, select your notebook instance by choosing <strong>Open JupyterLab</strong> for the JupyterLab interface.</li> 
 <li>Select the <em>Upload Files</em> icon as shown in the following screenshot:<img alt="Upload files" class="wp-image-5424 size-full" height="167" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/07/14/sagemaker_upload_icon.png" width="750" /></li> 
 <li>Select this <a href="https://github.com/aws-samples/detect-scene-changes-aws-iot-events/blob/main/sagemaker_model/iot-notebook.ipynb">this notebook</a> (previously downloaded) in your file system.</li> 
 <li>Once the file is uploaded, select the <em>Create a new folder</em> icon (as shown in the following screenshot), and set the folder name to “images.”<img alt="Create a new folder" class="wp-image-5425 size-full" height="167" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/07/14/sagemaker_folder_icon.png" width="750" /></li> 
 <li>Go into the images folder and upload images from this <a href="https://github.com/aws-samples/detect-scene-changes-aws-iot-events/tree/main/sagemaker_model/images">location</a>.</li> 
 <li>On the <em>Run</em> tab, select Run All Cells. Once done, your model endpoint should be visible in the Amazon SageMaker console.</li> 
 <li>Go to the Amazon SageMaker console.</li> 
 <li>Select <em>Inference</em> and and select <em>Endpoints</em> to confirm the availability of your model endpoint.</li> 
</ol> 
<h2><strong>To upload asset images in Amazon S3 and inspect them with AWS Lambda</strong></h2> 
<p>When a device captures a scene image, it is uploaded to an Amazon S3 bucket. Then,&nbsp;an AWS Lambda function gets triggered in order to evaluate the image against the Amazon SageMaker model. If changes are detected, the Lambda function publishes a notification to an&nbsp;Amazon Simple Notification Service (SNS) topic.</p> 
<p>In order to set up this solution, follow these steps:</p> 
<h3><strong>Amazon Simple Notification Service setup</strong>:</h3> 
<ol> 
 <li>Create a SNS topic. For more information, see the <a href="https://docs.aws.amazon.com/sns/latest/dg/">Amazon Simple Notification Service Developer Guide</a> and, more specifically, the documentation of the <a href="https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-topic.html">CreateTopic</a> operation in the Amazon Simple Notification Service API Reference.</li> 
 <li>Subscribe to a SNS topic by following the guide <a href="https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-subscribe-endpoint-to-topic.html#create-subscribe-endpoint-to-topic-aws-console">To Subscribe an Endpoint to an Amazon SNS Topic Using the AWS Management Console</a>.</li> 
</ol> 
<h3><strong>AWS Lambda setup</strong>:</h3> 
<ol> 
 <li>Create an IAM policy with the below JSON policy document. For more information on the procedure, please refer to the steps <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html#access_policies_create-json-editor">in the IAM user guide</a>.</li> 
</ol> 
<pre><code class="lang-json">{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": [
"sagemaker:InvokeEndpoint"
],
"Resource": [
"arn:aws:sagemaker:region:account_number:endpoint/#endpoint_name#"
]
},
{
"Effect": "Allow",
"Action": [
"sns:Publish*"
],
"Resource": [
"sns_topic_arn"
]
}
]
}
</code></pre> 
<ol> 
 <li> 
  <ul> 
   <li>Note: In the above policy, replace the content in <strong>bold</strong> with a valid <strong>region</strong>, <strong>account number</strong>, <strong>Amazon&nbsp;SageMaker endpoint name</strong>&nbsp;(Example: <em><strong>linear-learner-2021-01-01-22-39-49-363</strong></em>) and the&nbsp;<strong>SNS topic’s ARN</strong> (format: arn:aws:sns:<strong>region</strong>:<strong>account</strong>:<strong>snsTopicName</strong>)</li> 
  </ul> </li> 
 <li>Create an IAM role for your AWS Lambda function as described in this <a href="https://docs.amazonaws.cn/en_us/lambda/latest/dg/lambda-intro-execution-role.html#permissions-executionrole-console">user guide</a>. 
  <ul> 
   <li>When prompted to select permissions, choose the policy previously created.</li> 
   <li>Choose the Amazon managed policy <em>AmazonS3ReadOnlyAccess</em>.</li> 
  </ul> </li> 
 <li>Create an AWS Lambda function as described in <a href="https://docs.aws.amazon.com/lambda/latest/dg/getting-started-create-function.html">creating a Lambda function</a>. 
  <ul> 
   <li>For the Runtime, choose a Python version (Example: Python 3.8).</li> 
   <li>For the Role, choose the IAM previously created.</li> 
  </ul> </li> 
 <li>Once the AWS Lambda function is created, do the following: 
  <ul> 
   <li>Under the <em>Configuration</em> tab, go to <em>General Configuration</em>, select the <em>Edit</em> button to set the <em>Memory</em> to 256 (MB) and the <em>Timeout</em> to 1 minute. Then select the <em>Save</em> button.</li> 
   <li>Under the <em>Code</em> tab, go to the <em>Layers</em> section and select the <em>Add a layer</em> button.</li> 
   <li>Select AWS Layers in the <em>Layer source</em> field, choose the <em>AWS Lambda SciPy Layer for Python</em> option.</li> 
   <li>Select a version in the <em>Version</em> field and select the <em>Add</em> button.</li> 
  </ul> </li> 
 <li>Create a directory on your computer, download the AWS Lambda source code available at this <a href="https://github.com/aws-samples/detect-scene-changes-aws-iot-events/tree/main/lambda">location</a> and copy it in the directory created. 
  <ul> 
   <li>In this directory, open the lambda_function.py file and add a valid Amazon SageMaker Endpoint name (<strong>#SAGEMAKER_ENDPOINT_NAME#</strong>) and a valid SNS topic’s ARN (<strong>#SNS_TOPIC_ARN#</strong>). Once done, save the file.</li> 
   <li>Run the command: <em>pip3 install –upgrade -r requirements.txt -t . —no-dependencies</em></li> 
   <li>Select all the files and folders in this directory and create a zip file.</li> 
   <li>Note: The Pip version version must match the AWS Lambda’s Python version (Example: Pip 3.8 for Python 3.8).</li> 
  </ul> </li> 
 <li>In the AWS Lambda console, go to the Code Source section, select the <em>Upload From</em> button, select the <em>Upload</em> button, select your zip file on your computer, and then select the <em>Save</em> button.</li> 
</ol> 
<h3><strong>Amazon S3 setup</strong>:</h3> 
<ol> 
 <li>Create an Amazon S3 bucket as described in this <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-bucket.html">user guide</a>.</li> 
 <li>Create an Amazon S3 event notification in order to trigger an AWS Lambda function when a new image is uploaded in the Amazon S3 bucket. See the documentation to <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/enable-event-notifications.html">enable event notifications</a>: 
  <ul> 
   <li>In the Event types section, select the “<em>All object create events</em>” event type option (“s3:ObjectCreated:*”).</li> 
   <li>In the Destination section, select <em>Lambda function</em> as a Destination type and specify the ARN of the AWS Lambda function previously created.</li> 
  </ul> </li> 
</ol> 
<h2><strong>To set up AWS IoT Core for device connection</strong></h2> 
<h3><strong>Create device credentials</strong></h3> 
<p>You can create device credentials by following the guides below.&nbsp;Once completed, you will have a certificate, private key, public key and root CA certificate.</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-moisture-policy.html">Create an AWS IoT Core Policy</a></li> 
 <li><a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-moisture-create-thing.html">Create Device Credentials and attach IoT Core Policy to a Device Certificate</a>.</li> 
 <li>Attach the following policy to the device certificate.</li> 
</ol> 
<p>Note: In the following policy, replace the content in <strong>bold</strong> with a valid region and AWS account ID.</p> 
<pre><code class="lang-json">{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": "iot:Connect",
"Resource": "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:client/clientid"
},
{
"Effect": "Allow",
"Action": ["iot:Publish", "iot:Receive"],
"Resource": "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:topic/deviceInboundTopic"
},
{
"Effect": "Allow",
"Action": ["iot:Publish", "iot:Receive"],
"Resource": "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:topic/deviceOutboundTopic"
},
{
"Effect": "Allow",
"Action": ["iot:Subscribe"],
"Resource": "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:topicfilter/*"
}
]
}</code></pre> 
<h3><strong>Connect to AWS IoT Core</strong></h3> 
<p>Once the device credentials are created, you can now connect your IoT device to AWS IoT Core and publish the device notification by following the steps outlined.</p> 
<p>Note: you must install the Python SDK (visit <a href="https://aws.amazon.com/sdk-for-python/">this page</a>) and the AWS IoT Device SDK (visit <a href="https://github.com/aws/aws-iot-device-sdk-python-v2">this page</a>) to run the following code snippets.</p> 
<pre><code class="lang-python">import boto3
import json
import logging
import time
from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient
logging.basicConfig(filename='pythonIotDevice.log', filemode='w', format='%(name)s - %(levelname)s - %(message)s',level=logging.DEBUG)
logger = logging.getLogger('pythonIotDevice')
logger.info("pythonIotDevice")

#Connection to the AWS IoT Core with Root CA certificate and unique device credentials (keys and certificate) previously retrieved
# For certificate based connection
myMQTTClient = AWSIoTMQTTClient("clientid")

# For TLS mutual authentication
myMQTTClient.configureEndpoint("<strong><em>your.iot.endpoint</em></strong>", 8883) <em><strong>#Provide your AWS IoT Core endpoint (Example: "abcdef12345-ats.iot.us-east-1.amazonaws.com")</strong></em>
myMQTTClient.configureCredentials("<strong><em>/root/ca/path</em></strong>", "<strong><em>/private/key/path</em></strong>", "<strong><em>/certificate/path</em></strong>") <em><strong>#Set paths for Root CA, for private key and private certificate</strong></em>

myMQTTClient.configureOfflinePublishQueueing(-1)
myMQTTClient.configureDrainingFrequency(2)
myMQTTClient.configureConnectDisconnectTimeout(10)
myMQTTClient.configureMQTTOperationTimeout(5)
logger.info("Connecting...")
myMQTTClient.connect()

#Publish device notification to AWS IoT Core
deviceId = 1
timestamp = time.time();
logger.info("Publishing...")
myMQTTClient.publish("deviceInboundTopic", "{\"deviceID\":\"" + str(deviceId) + "\",\"notifType\":\"StartRoute\",\"dateTm\":\""+ str(timestamp) +"\"}", 0)

def handleResponse(client, userdata, message):
jsonMessage = json.loads(message.payload)
logger.info('jsonMessage=%s', jsonMessage)
logger.info("Subscribing...")
myMQTTClient.subscribe("deviceOutboundTopic", 1, handleResponse);

#Wait until reception of subscription confirmation (wait 60 seconds)
time.sleep(60)
logger.info("Disconnecting...")
myMQTTClient.disconnect()</code></pre> 
<h3>Connect to Amazon S3</h3> 
<p>In order for your device to upload images directly into Amazon S3,&nbsp;AWS IoT Core has a credentials provider that allows you to use the built-in <a href="https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html">X.509 certificate</a> as the unique device identity to authenticate AWS requests.&nbsp;This eliminates the need to store an access key ID and a secret access key on your device.</p> 
<p>To set it up, please refer to the <a href="https://docs.aws.amazon.com/iot/latest/developerguide/authorizing-direct-aws.html">documentation&nbsp;</a>for further information on the procedure. Attach the following IAM policy to the IAM role&nbsp;assumed by the credentials provider.</p> 
<pre><code class="lang-json">{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": "s3:PutObject",
"Resource": "arn:aws:s3:::<strong>#bucket_name#</strong>/*"
}
]
}</code></pre> 
<p>Note: Replace the content in <strong>bold</strong> with the bucket name previously created.</p> 
<p>The following code snippet only shows how to upload an image to Amazon S3. To run the code snippet, download one of the images from <a href="https://github.com/aws-samples/detect-scene-changes-aws-iot-events/tree/main/sagemaker_model/images">this location</a>. Then, specify its name and path as indicated below:</p> 
<pre><code class="lang-python">import boto3
import json
import logging
import time
import requests

logging.basicConfig(filename='pythonS3IotDevice.log', filemode='w', format='%(name)s - %(levelname)s - %(message)s',level=logging.DEBUG)
logger = logging.getLogger('pythonS3IotDevice')
logger.info("pythonS3IotDevice")

iot_credentials_endpoint='https://<strong><em>#iot_credentials_provider_endpoint#</em></strong>/role-aliases/<strong><em>#role_alias#</em></strong>/credentials' <em><strong>#Provide your AWS IoT Credentials endpoint (Example: "acbcdef12345.credentials.iot.us-east-1.amazonaws.com") and the IAM role alias previously created.</strong></em>

response = requests.get(iot_credentials_endpoint, cert=("<strong><em>/certificate/path</em></strong>", "<strong><em>/private/key/path</em></strong>")) <em><strong>#Set paths for private certificate and private key</strong></em>

if response:
tmp_credentials = response.json()
access_key_id = tmp_credentials['credentials']['accessKeyId']
secrete_access_key = tmp_credentials['credentials']['secretAccessKey']
session_token = tmp_credentials['credentials']['sessionToken']

s3client = boto3.client(
's3',
aws_access_key_id=access_key_id,
aws_secret_access_key=secrete_access_key,
aws_session_token=session_token,
)

s3client.upload_file('<strong>/jpegimage/path</strong>', "<strong>#bucket_name#</strong>", '<strong>#image_name#</strong>.jpg') <strong><em>#Set path for jpeg image to upload. Then specify the bucket name (name of the bucket previously created) and the image name.</em></strong></code></pre> 
<p>Note: For further AWS IoT SDK (for Python) samples, please visit <a href="https://github.com/aws/aws-iot-device-sdk-python-v2/tree/master/samples">this page</a>.</p> 
<h2>Clean up</h2> 
<p>After you are done, if you want to ensure that no additional cost is incurred, you can remove the resources provisioned in your account. This includes the deletion of the following resources:</p> 
<ol> 
 <li>Delete the AWS Lambda function with the following AWS CLI command: aws lambda delete-function —function-name <em><strong>#lambda_function_name#</strong></em> —region <em><strong>#region#</strong></em></li> 
 <li>Delete the Amazon S3 bucket as explained in <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/delete-bucket.html">this guide</a>.</li> 
 <li>Clean up the Amazon SageMaker resources (model, endpoint and notebook) as explained in <a href="https://docs.aws.amazon.com/sagemaker/latest/dg/ex1-cleanup.html">this guide</a>.</li> 
 <li>Delete the detector model created in AWS IoT Events with the following AWS CLI command: <em>aws iotevents delete-detector-model —detector-model-name <strong>#detector_model_name#</strong> —region <strong>#region#</strong></em></li> 
</ol> 
<p>Note that for the above AWS CLI commands:</p> 
<ul> 
 <li>The <em><strong>#lambda_function_name#</strong></em> is the name of the AWS Lambda’s function previously created.</li> 
 <li>The <em><strong>#region#</strong></em> is the AWS region where your resources were created (Example: us-east-1).</li> 
 <li>The <em><strong>#detector_model_name#</strong></em> is the name of the detector model previously created in AWS IoT Events.</li> 
</ul> 
<h2>Conclusion</h2> 
<p>In this post, we walked through a use case where drone devices are used to capture scene images of a physical asset (e.g. gas pipe), which are then validated against a machine learning model to detect changes to the asset that may affect the proper operation of the asset.</p> 
<p>To summarize the solution steps:</p> 
<ol> 
 <li>First, we created a detector model in AWS IoT Events, that manages the drone device route.</li> 
 <li>Second, we provisioned a sample machine learning model that validates the images received from the device. We also created an Amazon S3 bucket to upload the images captured by the device.</li> 
 <li>Then we provisioned an AWS Lambda function to retrieve the images uploaded in the Amazon S3 bucket and send them to the Amazon SageMaker model for validation. We also created an Amazon Simple Notification System (SNS) topic to notify the end-users, if scene changes were detected by the Amazon SageMaker model.</li> 
 <li>Finally, we explained how the device can connect with AWS services (AWS IoT Core and Amazon S3) by providing AWS configuration procedures as well as Python code samples for the device.</li> 
</ol> 
<p>For further information on the services used, you can consult the&nbsp;<a href="https://aws.amazon.com/iot-core/">AWS IoT Core</a>,&nbsp;<a href="https://aws.amazon.com/iot-events/">AWS IoT Events</a> and <a href="https://aws.amazon.com/sagemaker/">Amazon SageMaker</a>&nbsp;web pages.</p>
