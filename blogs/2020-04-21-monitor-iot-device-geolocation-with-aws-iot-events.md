---
title: "Monitor IoT device geolocation with AWS IoT Events"
url: "https://aws.amazon.com/blogs/iot/monitor-iot-device-geolocation-with-aws-iot-events/"
date: "Tue, 21 Apr 2020 02:47:48 +0000"
author: "Mehdi Amrane"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-events/feed/"
---
<p>Organizations with large numbers of IoT devices need efficient solutions to track events occurring across multiple devices, in order to identify operational issues and act upon them. This post covers the use case of a fictitious organization AcmeTracker that offers services to track the geolocation of a wide suite of assets (such as vehicles), by using IoT devices to monitor and notify support teams when an asset is outside of its expected geolocation boundaries. The organization AcmeTracker uses AWS IoT services to manage the devices and its data. Each device gets unique device credentials (certificates and private keys) by using the&nbsp;Fleet Provisioning feature in AWS IoT Core, and each device geolocation is monitored by AWS IoT Events.</p> 
<p>In this post, we provide an operational overview of the aforementioned asset monitoring solution, and then describe how to setup the applicable AWS IoT services:</p> 
<ul> 
 <li>Setup AWS IoT Events to monitor GPS coordinates</li> 
 <li>Setup unique device credentials with Fleet Provisioning in AWS IoT Core</li> 
</ul> 
<h2>Solution overview</h2> 
<p>Consider a scenario where a fleet of vehicles must follow a specific itinerary. The geolocation input is used to monitor each vehicle and notify the vehicle operators if the vehicle is not following the expected itinerary.&nbsp;The organization AcmeTracker provides IoT devices manufactured with embedded provisioning claim credentials (certificates and keys).</p> 
<p>The devices use the provisioning claim credentials to authenticate with AWS IoT using the AWS IoT Device SDK for Python. Programming languages other than Python are supported by the AWS IoT Device SDK, and for the full list of programming languages supported by the AWS IoT Device SDK, visit this <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-sdks.html">page</a>.</p> 
<p>This asset monitoring solution involves the following&nbsp;sequence of data and message exchanges:</p> 
<ol> 
 <li>The device requests the creation of unique device credentials (create certificate and keys) to the AWS IoT Core service via a MQTT topic.</li> 
 <li>The device requests to register itself (activate unique device credentials) in the AWS IoT Core service via a MQTT topic, based on a provisioning template defined in the AWS IoT Core service.</li> 
 <li>The device retrieves GPS coordinates from a satellite.</li> 
 <li>The device publishes its GPS coordinates to the AWS IoT Core service over a MQTT topic.</li> 
 <li>The <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-rules.html">AWS IoT Core rules engine</a> retrieves the GPS coordinates from the MQTT topic.</li> 
 <li>The AWS IoT Core rules engine sends the GPS coordinates to the AWS IoT Events service.</li> 
 <li>The <a href="https://aws.amazon.com/iot-events/">AWS IoT Events</a> service has a detector model that monitors incoming IoT events (GPS coordinates) to detect if a device is in its expected boundaries.</li> 
 <li>If a device state changes (either in boundary or out of boundary), the detector model sends a message to an <a href="https://aws.amazon.com/sns/">Amazon Simple Notification Service (SNS) topic</a>.</li> 
 <li>The end-users subscribed to the SNS topic receive a notification message to inform them of the device’s state change.</li> 
</ol> 
<p>Note: Step 1 and Step 2 are only applicable for new devices for which unique device credentials are not yet generated and activated. Devices that have unique device credentials start the flow from Step 3.</p> 
<p><img alt="Setup AWS IoT Events to monitor GPS coordinates" class="aligncenter wp-image-3844" height="511" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2020/04/18/ArduinoBlogIOT-2-6-2-12-5-Copy-of-Copy-of-Page-1.png" width="605" /></p> 
<p>In order to monitor and respond to changing GPS coordinates of the IoT device (covered in steps 6-9 above), AWS IoT Events must be configured accordingly. This is reviewed in the section below named&nbsp;<strong>Setup AWS IoT Events to monitor GPS coordinates</strong>.</p> 
<p>For the IoT devices to connect&nbsp;to AWS IoT Core (covered in steps&nbsp;1, 2, 4 and 5 above), the Fleet Provisioning feature must be enabled. This is reviewed in the section below named&nbsp;<strong>Setup unique device credentials with Fleet Provisioning in AWS IoT Core</strong>.</p> 
<p>Note: This solution implementation requires the selection of an AWS Region where Amazon Simple Notification Service (SNS), AWS IoT Core, and AWS IoT Events services are available. Visit the AWS Region table <a href="https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/">here</a>&nbsp;for a full list of AWS Regions where these AWS services are available.</p> 
<h2>Setup AWS IoT Events to monitor GPS coordinates</h2> 
<p>In AWS IoT Events, the organization AcmeTracker creates the below components to implement the asset monitoring solution:</p> 
<ul> 
 <li>One input representing the data, sent by the IoT device to the AWS Cloud, which includes the geolocation details (latitude and longitude)</li> 
 <li>A detector model with 2 states (and 2 transitions) to detect if the device is in boundary or out of boundary. The detector model uses the input data to detect the state associated to a device.</li> 
</ul> 
<p>The following subsections explains how&nbsp;the above components are implemented and how to configure AWS IoT Core to forward the device geolocation data to AWS IoT Events.</p> 
<h3>Create Input in AWS IoT Events</h3> 
<p>You can create an input in AWS IoT Events by following the guide to <a href="https://docs.aws.amazon.com/pt_br/iotevents/latest/developerguide/iotevents-detector-input.html">create an input</a>. In our example, the organization AcmeTracker creates an input with the following details:</p> 
<ul> 
 <li>An Input name set to “gpsInput”</li> 
 <li>An example JSON payload/event with the content below</li> 
</ul> 
<pre><code class="lang-json">{
  "gpsDeviceID": "1",
  "gpsLat": 10.1,
  "gpsLng": 10.1,
  "gpsDatTm": "03/02/2020&nbsp; 08:03:20.200"
}</code></pre> 
<h3>Create and publish Detector Model in AWS IoT Events</h3> 
<p>In our example, the organization AcmeTracker creates a detector model with the following details:</p> 
<ul> 
 <li>Two states (InBoundary and OutOfBoundary)</li> 
 <li>Two transitions (InBoundaryTransition and OutOfBoundaryTransition) in order to move the device from 1 state to another</li> 
</ul> 
<p>The detector model design is illustrated below:</p> 
<p><img alt="detector model" class="alignnone wp-image-3870 size-large" height="373" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2020/04/18/aws_iot_events_state_machine-1024x373.png" width="1024" /></p> 
<p>To create a detector model,&nbsp;you can download this <a href="https://github.com/aws-samples/aws-monitor-device-geolocation-iotevents/blob/master/detector-model-sample.json">sample detector model file</a>&nbsp;and import it into AWS IoT Events by following the steps below:</p> 
<ol> 
 <li>Create a SNS topic. For more information, see the <a href="https://docs.aws.amazon.com/sns/latest/dg/">Amazon Simple Notification Service Developer Guide</a> and, more specifically, the documentation of the <a href="https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-topic.html">CreateTopic</a> operation in the Amazon Simple Notification Service API Reference.</li> 
 <li>Subscribe to a SNS topic by following the guide <a href="https://docs.aws.amazon.com/sns/latest/dg/sns-tutorial-create-subscribe-endpoint-to-topic.html#create-subscribe-endpoint-to-topic-aws-console">To Subscribe an Endpoint to an Amazon SNS Topic Using the AWS Management Console</a>.</li> 
 <li>Create an IAM role for the Detector Model. For more information, see the documentation for <a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/iotevents-start.html#iotevents-permissions-console">setting up permissions for AWS IoT Events</a>.</li> 
 <li>Download the sample detector model file from the repository at this <a href="https://github.com/aws-samples/aws-monitor-device-geolocation-iotevents/blob/master/detector-model-sample.json">location</a>.</li> 
 <li>Update the file with the SNS topic’s ARN (replace value “#snsTopicArn#” with a value in the following format: arn:aws:sns:<strong>region:account:snsTopicName</strong>)</li> 
 <li>Update the file with the Detector Model IAM role’s ARN (replace value “#detectorModelRoleArn#”with a value in the following format: arn:aws:iam::<strong>account</strong>:role/service-role/<strong>detectorRoleName</strong>)</li> 
 <li>Go to the AWS console and select the AWS IoT Events service</li> 
 <li>Click on Create detector model and click on Import detector model</li> 
 <li>Click on Import, select the file from your local system and click on Open</li> 
 <li>Once done, the Detector Model is created</li> 
 <li>Once the Detector Model is created, you can publish it by following the guide <a href="https://docs.aws.amazon.com/pt_br/iotevents/latest/developerguide/iotevents-detector-model.html">Create a Detector Model</a>. As the organization AcmeTracker needs to track the states of multiple devices, the detector generation model is set to “Create a detector for each key value” with a key set to the gpsDeviceID. The Detector evaluation model is set to “Batch evaluation”.</li> 
</ol> 
<h3>Boundary State Configuration in Detector Model</h3> 
<p>In our example, the organization AcmeTracker sets up the detector model’s states to notify a SNS topic after evaluating the input received. If the input received shows that the device is in bounds or out of bounds, a notification is sent to a SNS topic to inform the end user of the current device state (respectively “InBoundary” or “OutOfBoundary”).</p> 
<p>The points below outline the configuration of the 2 states:</p> 
<ul> 
 <li>Both states include an OnEnter event and an OnInput event sending a notification to a SNS topic based on an event condition</li> 
 <li>Example of event condition for the “InBoundary” state:</li> 
</ul> 
<p><em><code>$input.gpsInput.gpsLat&gt;=10.1 &amp;&amp; $input.gpsInput.gpsLat&lt;=10.2 &amp;&amp; $input.gpsInput.gpsLng&gt;=10.1 &amp;&amp; $input.gpsInput.gpsLng&lt;=10.2</code></em></p> 
<ul> 
 <li>Example of event condition for the “OutOfBoundary” state:</li> 
</ul> 
<p><em><code>$input.gpsInput.gpsLat&lt;10.1 || $input.gpsInput.gpsLat&gt;10.2 || $input.gpsInput.gpsLng&lt;10.1 || $input.gpsInput.gpsLng&gt;10.2</code></em></p> 
<p>Note: The above coordinates (latitude and longitude) are only illustrative.</p> 
<h3>Boundary Transition Configuration in Detector Model</h3> 
<p>In our example, the organization AcmeTracker sets up transitions in the detector model in order to change the state of the device after evaluating the device’s input against a condition (Event trigger logic).</p> 
<p>The below points outline the configuration of each transition:</p> 
<ul> 
 <li>Both transitions include an event trigger logic which is used as a condition to move from one state to the other</li> 
 <li>Example of Event trigger logic for “InBoundaryTransition” transition:</li> 
</ul> 
<p><em><code>$input.gpsInput.gpsLat&gt;=10.1 &amp;&amp; $input.gpsInput.gpsLat&lt;=10.2 &amp;&amp; $input.gpsInput.gpsLng&gt;=10.1 &amp;&amp; $input.gpsInput.gpsLng&lt;=10.2</code></em></p> 
<ul> 
 <li>Example of Event trigger logic for “OutOfBoundaryTransition” transition:</li> 
</ul> 
<p><em><code>$input.gpsInput.gpsLat&lt;10.1 || $input.gpsInput.gpsLat&gt;10.2 || $input.gpsInput.gpsLng&lt;10.1 || $input.gpsInput.gpsLng&gt;10.2</code></em></p> 
<p>Note: The above coordinates (latitude and longitude) are only illustrative.</p> 
<h3>Configure AWS IoT Core rules</h3> 
<p>An AWS IoT Core rule needs to be configured to forward device geolocation data from AWS IoT Core (MQTT topic) to AWS IoT Events.</p> 
<ul> 
 <li>Go to the AWS console and select the AWS IoT Core service</li> 
 <li>Click on <em>Act, Rules</em> and then click <em>Create a Rule</em></li> 
 <li>Set the Name for the rule and Set the Rule query statement to <code><em>SELECT * FROM ‘<strong>gpsTopic</strong>’</em></code></li> 
 <li>Click on “<em>Add action”</em>, <em>Send a message to an IoT Events Input</em> and <em>Configure action</em></li> 
 <li>Select the Input previously created</li> 
 <li>Click on <em>Create Role</em> and enter a role name</li> 
 <li>Click on <em>Add action</em></li> 
 <li>Click on <em>Create rule</em></li> 
</ul> 
<h2>Setup unique device credentials with Fleet Provisioning in AWS IoT Core</h2> 
<p>The organization AcmeTracker uses the Fleet Provisioning feature in AWS IoT Core to enable a fleet of IoT devices sharing the same business function to be managed with the same policies and the same AWS IoT Core rules. In our example and as outlined in the solution overview, the organization AcmeTracker provides IoT devices manufactured with embedded provisioning claim credentials (certificates and keys). The devices use the provisioning claim credentials to authenticate with AWS IoT Core using the AWS IoT Device SDK for Python. By using Fleet Provisioning, the IoT devices can exchange these credentials with unique device credentials for regular operations. This section explains how to create provisioning claim credentials, create a provisioning template and connect to AWS IoT Core.</p> 
<h3>Create provisioning claim credentials</h3> 
<p>You can create provisioning claim credentials by following the guides below.&nbsp;Once completed, you will have a certificate, private key, public key and root CA certificate.</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/iot/latest/developerguide/create-device-certificate.html">Create and Activate a Device Certificate</a></li> 
 <li><a href="https://docs.aws.amazon.com/iot/latest/developerguide/create-iot-policy.html">Create an AWS IoT Core Policy</a></li> 
 <li><a href="https://docs.aws.amazon.com/iot/latest/developerguide/attach-policy-to-certificate.html">Attach an AWS IoT Core Policy to a Device Certificate</a>. Attach the following policy to the device certificate:</li> 
</ol> 
<p>Note: In the policy below, replace the region and account variables (<strong>region:account</strong>) with a valid region and account number (Example: <strong>us-east-1:123456789</strong>).</p> 
<pre><code class="lang-json">{
&nbsp; "Version": "2012-10-17",
&nbsp; "Statement": [
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Effect": "Allow",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Action": "iot:Connect",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Resource": "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:client/<em>clientid</em>"
&nbsp;&nbsp;&nbsp; },
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Effect": "Allow",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Action": ["iot:Publish","iot:Receive"],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Resource": [
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:topic/$aws/certificates/create/*",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:topic/$aws/provisioning-templates/<em>iotDeviceTemplateName</em>/provision/*"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ]
&nbsp;&nbsp;&nbsp; },&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Effect": "Allow",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Action": ["iot:Subscribe"],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Resource": [
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:topicfilter/$aws/certificates/create/*",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:topicfilter/$aws/provisioning-templates/<em>iotDeviceTemplateName</em>/provision/*"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ]
&nbsp;&nbsp;&nbsp; },
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Effect": "Allow",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Action": [
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "iot:CreateProvisioningClaim"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "Resource": [
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:provisioningtemplate/<em>iotDeviceTemplateName</em>"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ]
&nbsp;&nbsp;&nbsp; }
&nbsp; ]
}</code></pre> 
<h3>Create a provisioning template in AWS IoT Core</h3> 
<p>You can create a provisioning template in AWS IoT Core by following the steps below:</p> 
<ul> 
 <li>Go the AWS console and select AWS IoT Core service</li> 
 <li>Click on <em>Onboard</em>, <em>Fleet Provisioning templates</em> and then <em>Create</em></li> 
</ul> 
<p><img alt="AWS IoT Fleet Provisioning templates" class="aligncenter wp-image-3867 size-large" height="307" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2020/04/18/blog_solution_diagram3-1024x307.png" width="1024" /></p> 
<ul> 
 <li>Click on Get Started</li> 
 <li>Choose a template name (in our example: <em><strong>iotDeviceTemplateName</strong></em>)</li> 
 <li>In the Provisioning role, click on <em>Create a Role</em></li> 
 <li>Then click on <em>Next</em>, click on <em>Advanced mode</em> and copy paste an AWS IoT policy with the following details:</li> 
</ul> 
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
      "Action": ["iot:Publish","iot:Receive"],
      "Resource": "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:topic/gpsTopic"
    },
    {
      "Effect": "Allow",
      "Action": ["iot:Subscribe"],
      "Resource": "arn:aws:iot:<strong>region</strong>:<strong>account</strong>:topicfilter/*"
    }
  ]
}</code></pre> 
<ul> 
 <li>Click on <em>Create template</em></li> 
 <li>Click on <em>Enable template</em> (you do not need to select any certificate in the “Use provisioning claim” section)</li> 
</ul> 
<h3>Connect to AWS IoT Core</h3> 
<p>Once the provisioning template is enabled, you can now connect your IoT device to AWS IoT Core and publish the GPS coordinates by following the steps outlined below.</p> 
<p>Note that you must install the Python SDK (visit <a href="https://aws.amazon.com/sdk-for-python/">this page</a>) and the AWS IoT Device SDK (visit <a href="https://github.com/aws/aws-iot-device-sdk-python-v2">this page</a>) to run the below snippets.</p> 
<ul> 
 <li><strong>Step 1:</strong> Create Keys and Certificate by using the below code snippet:</li> 
</ul> 
<pre><code class="lang-python">import boto3
import json
import logging
import time
from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient

logging.basicConfig(filename='pythonIotDeviceCreate.log', filemode='w', format='%(name)s - %(levelname)s - %(message)s',level=logging.DEBUG)
logger = logging.getLogger('pythonIotDevice')
logger.info("pythonIotDevice")

global certificateOwnershipToken
global certificatePem
global privateKey
certificateOwnershipToken = ''
certificatePem = ''
privateKey = ''

#Connection to the AWS IoT Core with Root CA certificate and provisioning claim credentials (private key and certificates)

# For certificate based connection
myMQTTClient = AWSIoTMQTTClient("clientid")
# For TLS mutual authentication
myMQTTClient.configureEndpoint("your.iot.endpoint", 8883) #Provide your AWS IoT Core endpoint (Example: "abcdef12345-ats.iot.us-east-1.amazonaws.com")
myMQTTClient.configureCredentials("/root/ca/path", "/private/key/path", "/certificate/path") #Set path for Root CA and provisioning claim credentials
myMQTTClient.configureOfflinePublishQueueing(-1)
myMQTTClient.configureDrainingFrequency(2)
myMQTTClient.configureConnectDisconnectTimeout(10)
myMQTTClient.configureMQTTOperationTimeout(5)
 
logger.info("Connecting...")
myMQTTClient.connect()

#Create keys and certificate by publishing a request in a MQTT topic
 
logger.info("Publishing...")
myMQTTClient.publish("$aws/certificates/create/json", "{}", 0)

#Subscribe to a separate MQTT topic to retrieve the unique device credentials and certificate ownership token

def certCallback(client, userdata, message):
jsonMessage = json.loads(message.payload)
certificateOwnershipToken = jsonMessage['certificateOwnershipToken']
certificatePem = jsonMessage['certificatePem']
privateKey = jsonMessage ['privateKey']
logger.info('certificateOwnershipToken=%s, certificatePem=%s, privateKey=%s', certificateOwnershipToken, certificatePem, privateKey)

logger.info("Subscribing...")
myMQTTClient.subscribe("$aws/certificates/create/json/accepted", 1,  certCallback);

#Wait until reception of subscription confirmation (sleep time set to 60 seconds)
time.sleep(60)

logger.info("Disconnecting...")
myMQTTClient.disconnect()</code></pre> 
<ul> 
 <li><strong>Step 2:</strong> Open the log file generated (pythonIotDeviceCreate.log). Then retrieve the certificate ownership token and unique device credentials (private key and certificate) as they will be necessary for the next steps. Finally,&nbsp;&nbsp;save the private key and certificate in separate files.</li> 
 <li><strong>Step 3:</strong> Register device by using the below code snippet:</li> 
</ul> 
<pre><code class="lang-python">#Register the device (link the unique device credentials with the provisioning template) by publishing a request in a MQTT topic with the certificate ownership token

import boto3
import json
import logging
import time
from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient

logging.basicConfig(filename='pythonIotDeviceRegister.log', filemode='w', format='%(name)s - %(levelname)s - %(message)s',level=logging.DEBUG)
logger = logging.getLogger('pythonIotDevice')
logger.info("pythonIotDevice")

#Connection to the AWS IoT Core with Root CA certificate and provisioning claim credentials (private key and certificates)
 
# For certificate based connection
myMQTTClient = AWSIoTMQTTClient("clientid")
# For TLS mutual authentication
myMQTTClient.configureEndpoint(<strong>"your.iot.endpoint"</strong>, 8883) 
<strong>#Provide your AWS IoT Core endpoint (Example: "abcdef12345-ats.iot.us-east-1.amazonaws.com") </strong>myMQTTClient.configureCredentials(<strong>"/root/ca/path", "/private/key/path", "/certificate/path"</strong>) 
<strong>#Set path for Root CA and provisioning claim credentials (do not use the private key and certificate retrieved from the logs in Step 1 since those credentials are not yet activated) </strong>myMQTTClient.configureOfflinePublishQueueing(-1)
myMQTTClient.configureDrainingFrequency(2)
myMQTTClient.configureConnectDisconnectTimeout(10)
myMQTTClient.configureMQTTOperationTimeout(5)

logger.info("Connecting...")
myMQTTClient.connect()

jsonInput = {
    "certificateOwnershipToken": <strong>"#certificateOwnershipToken#", #Provide the Certificate Ownership Token previously retrieved from the logs in Step 1</strong>
    "parameters": {
        "SerialNumber": <strong>"Provide-A-Device-Serial-Number" #Provide a Serial Number (Example: 012)</strong>
    }
}
 
logger.info("Publishing...")
myMQTTClient.publish("$aws/provisioning-templates/iotDeviceTemplateName/provision/json", json.dumps(jsonInput), 0)

#Subscribe to a separate MQTT topic to retrieve the confirmation

def templateCallback(client,  userdata, message):
    logger.info("Confirmation received: ")
    logger.info(message.payload)
    logger.info("from topic: ")
    logger.info(message.topic)

myMQTTClient.subscribe("$aws/provisioning-templates/iotDeviceTemplateName/provision/json/accepted", 1, templateCallback);

#Wait until reception of subscription confirmation (sleep time set to 60 seconds)
time.sleep(60)

logger.info("Disconnecting...")
myMQTTClient.disconnect()</code></pre> 
<ul> 
 <li><strong>Step 4:</strong> Publish GPS coordinates by using the below code snippet.</li> 
</ul> 
<pre><code class="lang-python">import boto3
import json
import logging
from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient

logging.basicConfig(filename='pythonIotDevicePublish.log', filemode='w', format='%(name)s - %(levelname)s - %(message)s',level=logging.DEBUG)
logger = logging.getLogger('pythonIotDevice')
logger.info("pythonIotDevice")

#Connection to the AWS IoT Core with Root CA certificate and unique device credentials (keys and certificate) previously retrieved

# For certificate based connection
myMQTTClient = AWSIoTMQTTClient("clientid")
# For TLS mutual authentication
myMQTTClient.configureEndpoint(<strong>"your.iot.endpoint"</strong>, 8883) <strong>#Provide your AWS IoT Core endpoint (Example: "abcdef12345-ats.iot.us-east-1.amazonaws.com")</strong>
myMQTTClient.configureCredentials(<strong>"/root/ca/path"</strong>, <strong>"/private/key/path"</strong>, <strong>"/certificate/path"</strong>) <strong>#Set path for Root CA and unique device credentials (use the private key and certificate retrieved from the logs in Step 1)</strong>
myMQTTClient.configureOfflinePublishQueueing(-1)
myMQTTClient.configureDrainingFrequency(2)
myMQTTClient.configureConnectDisconnectTimeout(10)
myMQTTClient.configureMQTTOperationTimeout(5)
 
logger.info("Connecting...")
myMQTTClient.connect()

#Publish gps coordinates to AWS IoT Core

myMQTTClient.publish("gpsTopic", "{\"gpsDeviceID\":\"1\",\"gpsLat\":10.1,\"gpsLng\":10.1,\"gpsDatTm\":\"03/02/2020  08:03:20.200\"}", 0)</code></pre> 
<ul> 
 <li><strong>Step 5:</strong> Verify that you received a notification, reflecting the device state (InBoundary), from the SNS topic you’ve previously subscribed to. You can also confirm that the device is in boundary in AWS IoT Events: Go to AWS IoT Events, click on your Detector model name and confirm the device’s current state is as illustrated below:</li> 
</ul> 
<p><img alt="This image shows the AWS IoT Events console where you click on your Detector model name and confirm the device’s current state" class="aligncenter wp-image-3868 size-large" height="430" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2020/04/18/aws_iot_events_in_boundary_state-1024x430.png" width="1024" /></p> 
<ul> 
 <li><strong>Step 6:</strong> Publish GPS coordinates by replacing the gps coordinates in the publish call of the previous code snippet with the below values:</li> 
</ul> 
<pre><code class="lang-python">myMQTTClient.publish("gpsTopic", "{\"gpsDeviceID\":\"1\",\<em><strong>"gpsLat\":10.0,\"gpsLng\":10.0</strong></em>,\"gpsDatTm\":\"03/02/2020  08:03:20.200\"}", 0)</code></pre> 
<ul> 
 <li><strong>Step 7:</strong> Verify that you received a notification, reflecting the device state (OutOfBoundary), from the SNS topic you’ve previously subscribed to. You can also confirm that the device is out of boundary in AWS IoT Events: Go to AWS IoT Events, click on your Detector model name and confirm the device’s current state is as illustrated below:</li> 
</ul> 
<p><img alt="Go to AWS IoT Events, click on your Detector model name and confirm the device’s current state is as illustrated in this image" class="aligncenter wp-image-3869 size-large" height="430" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2020/04/18/aws_iot_events_out_boundary_state-1024x430.png" width="1024" /></p> 
<p>Note: For further AWS IoT SDK (for Python) samples, please visit <a href="https://github.com/aws/aws-iot-device-sdk-python-v2/tree/master/samples">this page</a>.</p> 
<h2>Conclusion</h2> 
<p>In this post, we walked through a use case explaining how to monitor the geolocation of a IoT device fleet with the <a href="https://aws.amazon.com/iot-events/">AWS IoT Events</a> service, and how to create unique device credentials using the Fleet Provisioning feature to connect to <a href="https://aws.amazon.com/iot-core/">AWS IoT Core</a>.</p>
