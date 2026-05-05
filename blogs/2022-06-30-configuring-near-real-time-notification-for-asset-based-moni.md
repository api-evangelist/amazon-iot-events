---
title: "Configuring near real-time notification for asset based monitoring with AWS IoT Events"
url: "https://aws.amazon.com/blogs/iot/configuring-near-real-time-notification-for-asset-based-monitoring-with-aws-iot-events/"
date: "Thu, 30 Jun 2022 18:35:23 +0000"
author: "Prashanth Shankar Kumar"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-events/feed/"
---
<h2>Introduction</h2> 
<p>In today’s market, business success often lies in the ability to glean accurate insights and prediction. However, floor engineers and analysts have a hard time getting the information at the right instance to make an informed decision when there is a failure during operations.</p> 
<p>To make it easy to detect and respond to events from IoT sensors and applications, <a href="https://aws.amazon.com/iot-events/">AWS IoT Events</a> allows you to detect events across thousands of IoT sensors sending different telemetry data. Events are patterns of data that could help identify more complicated circumstances than expected, such as changes in equipment when a belt is stuck, or motion detectors using movement signals to activate lights and security cameras. AWS IoT Events allows you to simply select the relevant data sources to ingest, define the logic for each event using simple ‘if-then-else’ statements, and select the alert or custom action to trigger when an event occurs. It continuously monitors data from multiple IoT sensors and applications, and integrates with other services, such as AWS IoT Core, to enable early detection and unique insights into events.</p> 
<p>In this blog post, we explain how to design AWS IoT Events for two key issues faced by customers:</p> 
<p style="padding-left: 40px;">1. How to monitor and send alerts for assets at scale.</p> 
<p style="padding-left: 40px;">2. How to suppress repeated alerts during failures.</p> 
<h2>Solution overview</h2> 
<p>In this post, we describe the configuration of AWS IoT Events and <a href="https://aws.amazon.com/sns/?whats-new-cards.sort-by=item.additionalFields.postDateTime&amp;whats-new-cards.sort-order=desc">Amazon Simple Notification Service</a> (SNS) to send near real-time notification during failures, and cover data analysis using Amazon Athena.</p> 
<p><img alt="" class="alignnone size-full wp-image-8178" height="418" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/Picture1-1.png" width="879" /></p> 
<p style="text-align: center;"><em>&nbsp; Architecture for near real-time notification using AWS IoT Events</em></p> 
<h2>Solution walk through</h2> 
<h3>1) Prerequisites / Assumptions</h3> 
<p>We assume the following services are configured in your AWS environment, and devices are connected to AWS IoT core, for example a conveyor motor.</p> 
<p style="padding-left: 40px;">a) An on-premises edge gateway server where <a href="https://aws.amazon.com/greengrass/">AWS IoT Greengrass</a> service is running with required <a href="https://docs.aws.amazon.com/iot/latest/developerguide/x509-client-certs.html">AWS IoT certificates</a> and connected to AWS IoT Core as shown in the architecture. Additionally, your edge gateway sends data to the AWS IoT Core topic ‘iot/topic’ which can be routed to other permitted AWS services via AWS IoT Rules.</p> 
<p style="padding-left: 40px;">b) An <a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service</a> (Amazon S3) exists in your AWS account that you can use throughout this blog post. Throughout the blog post we assume the Amazon S3 bucket name is datalakes3bucket. We will use a sub-folder within that bucket that we call datastore. For security reasons we recommend turning on <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-encryption.html">server-side encryption</a> for your Amazon S3 bucket.</p> 
<p style="padding-left: 40px;">c) An existing Amazon SNS topic with subscription pointing to desired email or mobile number in your AWS account. Throughout the blog post we assume that the topic name is ‘iot/topic.’</p> 
<h3>2) Setting up the data lake</h3> 
<p>This section talks about setting up data analytics that can be used to run analytics on the stored data. Data received in AWS IoT Core on a topic in the message broker is then forwarded to <a href="https://aws.amazon.com/kinesis/data-firehose/">Amazon Kinesis Data Firehose</a>. Kinesis Data Firehose is an extract, transform, and load (ETL) service that reliably captures, transforms, and delivers streaming data to storage destinations like Amazon S3.</p> 
<p><strong>Step 1:</strong> Configure <a href="https://aws.amazon.com/kms/">AWS Key Management Service</a> (AWS KMS) for encrypting data at rest in the Kinesis Data Firehose delivery stream.</p> 
<p style="padding-left: 40px;">a)&nbsp; Sign in to the AWS Management Console</p> 
<p style="padding-left: 40px;">b)&nbsp; Search for <strong>AWS KMS</strong></p> 
<p style="padding-left: 40px;">c)&nbsp; &nbsp;Select <strong>Create new</strong></p> 
<p style="padding-left: 40px;">d)&nbsp; Choose the <strong>Key type</strong> to be <strong>Symmetric</strong></p> 
<p style="padding-left: 40px;">e)&nbsp; Leave the rest as default and create the key</p> 
<p style="padding-left: 40px;"><img alt="" class="alignnone size-full wp-image-8180" height="558" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/Picture2-1.png" width="937" /></p> 
<p style="text-align: left;"><em>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Creating a symmetric key in AWS Key Management Service</em></p> 
<p>This KMS key will be used while configuring Kinesis Data Firehose delivery stream in subsequent steps.</p> 
<p><strong>Step 2:</strong> Configure <a href="https://aws.amazon.com/cloudwatch/">Amazon CloudWatch</a> log group and log stream which will store the Kinesis Data Firehose log information.</p> 
<p style="padding-left: 40px;">a)&nbsp; &nbsp;Search <strong>Amazon CloudWatch</strong></p> 
<p style="padding-left: 40px;">b)&nbsp; &nbsp;Select<strong> Log groups</strong></p> 
<p style="padding-left: 40px;">c)&nbsp; &nbsp;Supply the <strong>log group name</strong>, in this blog post we will use &lt;example&gt;</p> 
<p style="padding-left: 40px;">d)&nbsp; Set the <strong>retention period</strong> according to your requirements</p> 
<p style="padding-left: 40px;"><img alt="" class="alignnone size-full wp-image-8186" height="489" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/14/Picture3-1.png" width="902" /></p> 
<p style="text-align: left;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<em> &nbsp; Setup Amazon CloudWatch log group for the Kinesis Data Firehose delivery stream</em></p> 
<p><strong>Step 3:</strong> Create a role with permissions in <a href="https://aws.amazon.com/iam/">Amazon Identity and Access Management</a> (IAM) that allows the Kinesis Data Firehose delivery stream to access the AWS KMS Key as well as the S3 bucket.</p> 
<p style="padding-left: 40px;">a)&nbsp; Search <strong>AWS IAM</strong></p> 
<p style="padding-left: 40px;">b)&nbsp; Select <strong>Policies</strong></p> 
<p style="padding-left: 40px;">c)&nbsp; Substitute below policy document</p> 
<p>Here and throughout this post, remember to replace the&nbsp;<span style="color: #ff0000;">placeholder account ID</span>&nbsp;with your own account ID, <span style="color: #ff0000;">region</span> with your own region, <span style="color: #ff0000;">s3 bucket</span> with your own bucket name.</p> 
<pre><code class="lang-json">{
   "Version": "2012-10-17",
   "Statement": [
     {
         "Sid": "",
         "Effect": "Allow",
         "Action": [
             "s3:AbortMultipartUpload",
             "s3:GetBucketLocation",
             "s3:GetObject",
             "s3:ListBucket",
             "s3:ListBucketMultipartUploads",
             "s3:PutObjectAcl",
             "s3:PutObject"
         ],
         "Resource": [
           "arn:aws:s3:::<span style="color: #ff0000;">datalakes3bucket</span>",
           "arn:aws:s3:::<span style="color: #ff0000;">datalakes3bucket</span>/<span style="color: #ff0000;">datastore</span>/*"
         ]
     },
     {
        "Sid": "",
        "Effect": "Allow",
        "Action": [
          "kms:GenerateDataKey",
          "kms:DescribeKey"
      ],
      "Resource": "arn:aws:kms:&lt;<span style="color: #ff0000;">region</span>&gt;:<span style="color: #ff0000;">123456789012</span>:key/&lt;<span style="color: #ff0000;">s3 bucket kms key</span>&gt;"
      },
      {
         "Sid": "",
         "Effect": "Allow",
         "Action": "logs:PutLogEvents",
         "Resource": "arn:aws:logs:&lt;<span style="color: #ff0000;">region</span>&gt;:<span style="color: #ff0000;">123456789012</span>:log-group:/aws/kinesisfirehose/kinesis-log-group:log-stream:kineis-log-stream"
      }
    ]
}</code></pre> 
<p>Next we will create the role to for the Kinesis Data Firehose delivery stream will assume. We also will attach the permission policy created above.</p> 
<p style="padding-left: 40px;">a)&nbsp; &nbsp;Select <strong>Roles</strong></p> 
<p style="padding-left: 40px;">b)&nbsp; Choose <strong>Create role</strong></p> 
<p style="padding-left: 40px;">c)&nbsp; &nbsp;Within the Use case selection menu, choose <strong>Kinesis</strong> and then <strong>Kinesis Firehose</strong></p> 
<p style="padding-left: 40px;">d)&nbsp; In the <strong>Permission policies</strong> selection field, search and choose the permission policy created above.</p> 
<p style="padding-left: 40px;">e)&nbsp; &nbsp;Give the role a name, leave the rest as default and choose <strong>Create role</strong></p> 
<p><strong>Step 4:</strong> Configure the Kinesis Data Firehose delivery stream to send data to the S3 bucket.</p> 
<p style="padding-left: 40px;">a)&nbsp; &nbsp;Search <strong>Amazon Kinesis</strong></p> 
<p style="padding-left: 40px;">b)&nbsp; &nbsp;Choose <strong>Create delivery stream</strong></p> 
<p style="padding-left: 40px;">c)&nbsp; &nbsp;Set source as <strong>Direct PUT</strong></p> 
<p style="padding-left: 40px;">d)&nbsp; Set destination to <strong>Amazon S3</strong></p> 
<p style="padding-left: 40px;">e)&nbsp; Set the <strong>Delivery stream name</strong></p> 
<p style="padding-left: 40px;">f)&nbsp; &nbsp;Select your pre-created S3 bucket in the <strong>Destination settings</strong></p> 
<p style="padding-left: 40px;"><img alt="" class="alignnone size-full wp-image-8275" height="608" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/23/Picture4-3.png" width="939" /></p> 
<p style="padding-left: 40px; text-align: center;"><em>Supply Destination settings</em></p> 
<p style="padding-left: 40px;">g)&nbsp; &nbsp;Go to <strong>Advanced settings</strong></p> 
<p style="padding-left: 40px;">h)&nbsp; Enable <strong>Amazon CloudWatch Error</strong> logging</p> 
<p style="padding-left: 40px;">i)&nbsp; Enable <strong>Server-Side encryption</strong> with the AWS KMS key you created</p> 
<p style="padding-left: 40px;">j)&nbsp; In the Permissions section, <strong>Choose existing IAM role</strong> and select the role we created in Step 3</p> 
<p style="padding-left: 40px;">k)&nbsp; Leave all the other options as default and choose <strong>Create delivery stream</strong></p> 
<p style="padding-left: 40px;"><img alt="" class="alignnone size-full wp-image-8276" height="943" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/23/Picture5-3.png" width="860" /></p> 
<p style="text-align: center;"><em>Configure the Advanced settings in the Kinesis Data Firehose delivery stream creation</em></p> 
<p><strong>Step 5:</strong> Setup an AWS IoT Core rule to filter data received on a topic (iot/topic) and route it to the Kinesis Data Firehose delivery stream. Please see the example data below that might arrive on the topic:</p> 
<pre><code class="lang-json">{
"Plant":
"demo",
"MachineName":
"conveyor-motor",
"EngineName":
"engine1",
"MessageTimestamp":
"2022-01-01",
"temperature": 90,
"amps": 15,
"Vibration": 100
}</code></pre> 
<p style="text-align: center;"><em>Sample Data send from the edge device to the AWS IoT Core message broker</em></p> 
<p style="padding-left: 40px;">a) In the console, search <strong>AWS IoT Core</strong> and select <strong>Rules</strong> under <strong>Act</strong></p> 
<p style="padding-left: 40px;">b) Select <strong>Create rule</strong> and provide it with a <strong>Name</strong> and <strong>Description</strong>. Choose <strong>Next</strong></p> 
<p><img alt="" class="alignnone size-full wp-image-8278" height="431" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/23/Picture6-3.png" width="879" /></p> 
<p style="text-align: center;"><em>Supply AWS IoT Core rule name</em></p> 
<p style="padding-left: 40px;">c) We use a SQL statement to filter the incoming data on the topic to only receive required fields from the messages. Copy and paste the below query statement into the field of the SQL statement</p> 
<p><code>SELECT MachineName, EngineName, MessageTimestamp, temperature, amps, Vibration FROM ‘iot/topic’</code></p> 
<p style="padding-left: 40px;">d) Choose <strong>Next</strong></p> 
<p style="padding-left: 40px;">e)&nbsp; In the section <strong>Rule actions</strong>, select <strong>Kinesis Firehose stream</strong>; search for the Kinesis Firehose stream we created prior</p> 
<p style="padding-left: 40px;">f) Choose <strong>Create new role</strong> in the Field below <strong>IAM role</strong> to create a new role with the according permissions</p> 
<p><strong>Step 6.</strong> We need to allow the Kinesis data firehose stream to use the AWS KMS key through adjusting the resource policy in AWS KMS.</p> 
<p style="padding-left: 40px;">a) Modify resource policy of the AWS KMS key associated with the Amazon S3 bucket to allow the Kinesis data firehose stream to use it</p> 
<pre><code class="lang-json">{
  "Version": "2012-10-17",
  "Id": "key-consolepolicy",
  "Statement": [
    {
      "Sid": "Allow Kinesis to use the key",
      "Effect": "Allow",
      "Principal": {
        "AWS": " arn:aws:iam::<span style="color: #ff0000;">123456789012</span>:role/firehoserole"
    },
    "Action": [
      "kms:GenerateDataKey*",
      "kms:DescribeKey"
    ],
    "Resource": " arn:aws:kms:&lt;<span style="color: #ff0000;">region</span>&gt;:<span style="color: #ff0000;">123456789012</span>:key/&lt;<span style="color: #ff0000;">s3 bucket kms key</span>&gt;"
    }
  ]
}</code></pre> 
<h3>3) Setup services for near real time alerting and notification</h3> 
<p><strong>Step 1.</strong> Let’s configure AWS IoT Events for near real-time configuration</p> 
<p style="padding-left: 40px;">a) Search<strong> AWS IoT Events</strong></p> 
<p style="padding-left: 40px;">b) Select <strong>Inputs</strong> and <strong>Create input</strong>. For this Blog Post we chose the Input name to be Alert_Input.</p> 
<p style="padding-left: 40px;">c)&nbsp; Below you find the content for the JSON file you need to create and upload as part of the Input configuration</p> 
<pre><code class="lang-json">{

"MachineName": "&lt;Machine Name/ID&gt;",

"EngineName": "&lt;Engine Name/ID&gt;",

"MessageTimestamp": "&lt;TimeStamp&gt;",

"temperature": &lt;temp&gt;,

"amps":&nbsp;&lt;amps&gt;,

"Vibration": &lt;vibration&gt;

}</code></pre> 
<p><img alt="" class="alignnone size-full wp-image-8279" height="502" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/23/Picture7-3.png" width="977" /></p> 
<p style="text-align: center;"><em>Input configuration within AWS IoT Events</em></p> 
<p><strong>Step 2.</strong> Let’s now setup the AWS IoT Rule to route incoming data to AWS IoT Events.</p> 
<p style="padding-left: 40px;">a) Search <strong>AWS IoT Core</strong> and select <strong>Rules</strong> under <strong>Act</strong></p> 
<p style="padding-left: 40px;">b) Enter the name for the AWS IoT Rule and choose <strong>Next</strong></p> 
<p style="padding-left: 40px;">c) Paste the SQL query to see below into the <strong>SQL statement</strong> field; replace the topic name placeholder in the FROM section with the topic where your data arrives</p> 
<p style="padding-left: 40px;">d) Choose <strong>Next</strong></p> 
<pre><code class="lang-sql">SELECT NodeID as MachineName, SubNodeID as EngineName, MessageTimestamp,&lt;br /&gt;&nbsp; get(get((SELECT Values FROM Data WHERE Name = 'engine1/temp'), 0).Values, 0).Value AS&lt;br /&gt;&nbsp; temperature, get(get((SELECT Values FROM Data WHERE Name = 'engine1/amps'), 0).Values, 0).Value AS amps,&lt;br /&gt;&nbsp; &nbsp;get(get((SELECT Values FROM Data WHERE Name = 'engine1/vib'), 0).Values, 0).Value AS Vibration FROM 'iot/topic'</code></pre> 
<p>In the above query we are selecting the columns names from the topic. The get() function used to read the values from the zeroth element from the nested json object.</p> 
<p style="padding-left: 40px;">e) As <strong>Rule actions</strong>, select <strong>IoT Event</strong> and choose the previously created <strong>Input name</strong> in the dropdown menu.</p> 
<p><strong>Step 3.</strong> In this section, you define a&nbsp;<a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/iotevents-detector-model.html">detector model</a>&nbsp;(a model of your equipment or process) using&nbsp;states. For each state, you define conditional (Boolean) logic that evaluates the incoming inputs to detect a significant event. When an event is detected, it changes the state and can trigger additional actions.</p> 
<p>In your states, you also define events that can execute actions whenever the detector enters or exits that state or when an input is received (these are known as&nbsp;<a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/iotevents-detector-model.html">OnEnter,&nbsp;OnExit&nbsp;and&nbsp;OnInput&nbsp;events</a>). The actions are executed only if the event’s conditional logic evaluates to&nbsp;true. We are going to see how to send notification when threshold is breached and also set timer between subsequent alert notifications. In the following steps we will go through all the states, their definitions as well as connections needed to create the whole detector model.</p> 
<p style="padding-left: 40px;">a) Search AWS IoT Events.</p> 
<p style="padding-left: 40px;">b) Select <strong>Create detector model</strong>, and then <strong>Create new</strong></p> 
<p style="padding-left: 40px;">c) The first detector state has been created for you. To modify it, select the circle with label State_1 in the main editing space and set the State name to “Running”</p> 
<p style="padding-left: 40px;"><img alt="" class="alignnone size-full wp-image-8283" height="495" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/23/Picture8-2.png" width="1097" /></p> 
<p style="text-align: center;"><em>Configuring the first AWS IoT Event state</em></p> 
<p style="padding-left: 40px;">d) In the field of <strong>OnEnter</strong> choose <strong>Add event</strong>. On the <strong>OnEnter</strong> event page, enter an <strong>Event name</strong> and the <strong>Event condition</strong>. The name of the event is <strong>init</strong> and the event condition is true. This indicates that the event is always triggered when the state is entered.</p> 
<p style="padding-left: 40px;">e) Choose <strong>Add action</strong></p> 
<ul> 
 <li> 
  <ul> 
   <li>Select <strong>Set variable</strong></li> 
   <li>For <strong>Variable operation</strong>, choose <strong>Assign value</strong></li> 
   <li>For <strong>Variable name,</strong> enter the name of the variable to set</li> 
   <li>For <strong>Variable value</strong>, enter the value 0 (zero)</li> 
  </ul> </li> 
</ul> 
<p style="padding-left: 40px;"><img alt="" class="alignnone size-full wp-image-8284" height="824" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/23/Picture9-2.png" width="866" /></p> 
<p style="text-align: center;"><em>Configuring OnEnter event for State Running</em></p> 
<p style="padding-left: 40px;">f) Choose <strong>Save</strong></p> 
<p style="padding-left: 40px;">g) Create another state by dragging and dropping State button. Modify the state name to Danger_OverVibration. In this state, we are evaluating if vibration exceeded threshold, if so then the action is set to send alert event to Amazon SNS</p> 
<p><img alt="" class="alignnone size-full wp-image-8285" height="422" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/23/Picture10-2.png" width="927" /></p> 
<p style="text-align: center;"><em>Configuring Danger_OverVibration State</em></p> 
<p style="padding-left: 40px;">h) Pause on the first state (<strong>Running</strong>). An arrow appears on the circumference of the state.</p> 
<p style="padding-left: 40px;">i) Click and drag the arrow from the first state (<strong>Running</strong>) to the second state (<strong>Danger_OverVibration</strong>). A directed line from the first state to the second state (labeled <strong>Untitled</strong>) appears.</p> 
<p style="padding-left: 40px;">j) Select the <strong>Untitled line</strong>. In the <strong>Transition event</strong> pane, enter an <strong>Event name</strong> and <strong>Event trigger logic</strong>.</p> 
<p><code> &nbsp;Event name: OverVibration</code></p> 
<p><code>&nbsp;Event Logic: $input.Alert_Input.Vibration &gt; <em><span style="color: #ff0000;">Threshold limit</span></em></code></p> 
<p><code>&nbsp;Event Actions:</code></p> 
<p><code>&nbsp; Set variable:</code></p> 
<p><code>&nbsp; &nbsp;Variable Operation: Assign Value</code></p> 
<p><code>&nbsp; &nbsp;Variable name: vibThresholdBreached</code></p> 
<p><code>&nbsp; &nbsp;Assign Value: $variable.vibThresholdBreached + 3</code></p> 
<p style="padding-left: 40px;">k) Choose <strong>Save</strong></p> 
<p style="padding-left: 40px;">l) Pause on the second state (<strong>Danger_OverVibration</strong>). An arrow appears on the circumference of the state.</p> 
<p style="padding-left: 40px;">m) Click and drag the arrow from the second state (<strong>Danger_OverVibration</strong>) to the first state (<strong>Running</strong>). A directed line from the second state to the first state (labeled <strong>Untitled</strong>) appears.</p> 
<p style="padding-left: 40px;">n) Select the <strong>Untitled</strong> line. In the <strong>Transition event</strong> pane, enter an <strong>Event name</strong> and <strong>Event trigger logic</strong>.</p> 
<p><code>&nbsp;Event name: VibBackToNormal</code></p> 
<p><code>&nbsp;Event Logic: $input.Alert_Input.Vibration &lt; <em><span style="color: #ff0000;">Threshold limit</span></em> &amp;&amp; $variable.vibThresholdBreached &lt;= 0</code></p> 
<p style="padding-left: 40px;">o) Choose <strong>Save</strong></p> 
<p style="padding-left: 40px;">p) In the field of <strong>OnEnter</strong> choose <strong>Add event</strong>. On the<strong> OnEnter</strong> event page, enter an <strong>Event name</strong> and the <strong>Event condition</strong>. The name of the event is <strong>Vibration_Breached</strong> and the event condition <strong>$variable.vibThresholdBreached &gt; 1</strong></p> 
<p style="padding-left: 80px;">1) Choose<strong> Add action</strong></p> 
<p><code>Event name: Vibration_Breached</code></p> 
<p><code>Event condition: $variable.vibThresholdBreached &gt; 1</code></p> 
<p><code>Event actions:</code></p> 
<p><code>&nbsp;Set Variable:</code></p> 
<p><code>&nbsp; Variable operation: Assign Value</code></p> 
<p><code>&nbsp; Variable name: initVibNotification</code></p> 
<p><code>&nbsp; Assign value: 1</code></p> 
<p style="padding-left: 80px;">2) Choose <strong>Save</strong></p> 
<p style="padding-left: 40px;">q. In the field of <strong>OnInput</strong> choose <strong>Add Event</strong>. We need to create an event condition to send email alert, then start a timer (5 mins) so that during this period subsequent email alerts will not be triggered, and finally trigger email when timeout ends. This process will continue until the vibration drops below threshold thereby ensuring email notification but only once within the time limit.</p> 
<p style="text-align: left; padding-left: 40px;">&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 1)&nbsp;&nbsp;&nbsp; Choose <strong>Add action</strong></p> 
<p style="text-align: left;"><code>Event name: Email Alert</code></p> 
<p style="text-align: left;"><code>Event condition: $variable.vibThresholdBreached &gt; 2 &amp;&amp; $variable.initVibNotification == 1</code></p> 
<p style="text-align: left;"><code>Event actions:</code></p> 
<p style="text-align: left;"><code>&nbsp;Set Timer:</code></p> 
<p style="text-align: left;"><code>&nbsp; Select operation: Create</code></p> 
<p style="text-align: left;"><code>&nbsp; Timer name: vibTimer</code></p> 
<p style="text-align: left;"><code>&nbsp; Enter duration: 5 Minutes (This is the time during which alert will be disabled)</code></p> 
<p style="text-align: left;"><code>&nbsp;Send SNS message</code></p> 
<p style="text-align: left;"><code>&nbsp; SNS Topic: arn:aws:sns: :&lt;<span style="color: #ff0000;">region</span>&gt;:<span style="color: #ff0000;">123456789012</span>:&lt;<span style="color: #ff0000;">Topic Name</span>&gt;</code></p> 
<p style="text-align: left;"><code>&nbsp; Select Default Payload</code></p> 
<p style="text-align: left;"><code>&nbsp; Payload Type: JSON</code></p> 
<p style="text-align: left;"><code>&nbsp;Set Variable:</code></p> 
<p style="text-align: left;"><code>&nbsp; Variable operation: Assign Value</code></p> 
<p style="text-align: left;"><code>&nbsp; Variable name: initVibNotification</code></p> 
<p style="text-align: left;"><code>&nbsp; Assign Value: 0</code></p> 
<p style="padding-left: 80px;">2)&nbsp; Choose <strong>Save</strong></p> 
<p style="padding-left: 80px;">3)&nbsp; Choose <strong>Add action</strong></p> 
<p><code>Event name: OverVibration</code></p> 
<p><code>Event condition: $input.Alert_Input.Vibration &gt; <span style="color: #ff0000;"><em>Threshold limit</em></span></code></p> 
<p><code>Event actions:</code></p> 
<p><code>&nbsp; Set Variable:</code></p> 
<p><code>&nbsp; &nbsp; Variable operation: Assign Value</code></p> 
<p><code>&nbsp; &nbsp; Variable name: vibThresholdBreached</code></p> 
<p><code>&nbsp; &nbsp; Assign Value: 3</code></p> 
<p style="padding-left: 80px;">4) Choose<strong> Save</strong></p> 
<p style="padding-left: 80px;">5) Choose <strong>Add action</strong></p> 
<p><code>Event name: TimerCheck</code></p> 
<p><code>Event condition: timeout("vibTimer")</code></p> 
<p><code>Event actions:</code></p> 
<p><code>&nbsp; Set Variable:</code></p> 
<p><code>&nbsp; &nbsp; Variable operation: Assign Value</code></p> 
<p><code>&nbsp; &nbsp; Variable name: initVibNotification</code></p> 
<p><code>&nbsp; &nbsp; Assign Value: 1</code></p> 
<p style="padding-left: 80px;">6)&nbsp; Choose <strong>Save</strong></p> 
<p style="padding-left: 80px;">7)&nbsp; Choose <strong>Add action</strong></p> 
<p><code>Event name: Normal</code></p> 
<p><code>Event condition: $input.Alert_Input.Vibration &lt; &lt;Threshold limit&gt;</code></p> 
<p><code>Event actions:</code></p> 
<p><code>&nbsp; Set Timer:</code></p> 
<p><code>&nbsp; &nbsp; Select operation: Destroy</code></p> 
<p><code>&nbsp; &nbsp; Timer name: vibTimer</code></p> 
<p><code>&nbsp; Set Variable:</code></p> 
<p><code>&nbsp; &nbsp; Variable operation: Decrement</code></p> 
<p><code>&nbsp; &nbsp; Variable name: vibThresholdBreached</code></p> 
<p><code>&nbsp; Set Variable:</code></p> 
<p><code>&nbsp; &nbsp; Variable operation: Assign Value</code></p> 
<p><code>&nbsp; &nbsp; Variable name: initVibNotification</code></p> 
<p><code>&nbsp; &nbsp; Assign Value: 1</code></p> 
<p style="padding-left: 80px;">8)&nbsp; Choose <strong>Save</strong></p> 
<p style="padding-left: 40px;">r)&nbsp; In the field of <strong>OnExit</strong> choose <strong>Add event.</strong> On the <strong>OnEnter</strong> event page, enter an <strong>Event name</strong> and the <strong>Event condition</strong>. The name of the event is <strong>Normal_Vibration_Restored</strong> and the event condition <strong>true</strong>.</p> 
<p style="padding-left: 80px;">a) Choose <strong>Add action</strong></p> 
<p><code>Event actions:</code></p> 
<p><code>&nbsp; Set SNS message:</code></p> 
<p><code>&nbsp; &nbsp; SNS Topic: arn:aws:sns: :&lt;<span style="color: #ff0000;">region</span>&gt;:<span style="color: #ff0000;">123456789012</span>:&lt;<span style="color: #ff0000;">Topic Name</span>&gt;</code></p> 
<p><code>&nbsp; &nbsp; Select Default Payload</code></p> 
<p><code>&nbsp; &nbsp; Payload Type: JSON</code></p> 
<p style="padding-left: 80px;">b)&nbsp; Choose <strong>Save</strong></p> 
<h3>4) Analyzing Data using Amazon Athena</h3> 
<p><a href="https://aws.amazon.com/glue/?whats-new-cards.sort-by=item.additionalFields.postDateTime&amp;whats-new-cards.sort-order=desc">AWS Glue</a> is a serverless data integration service that makes it easy to discover, prepare, and combine data for analytics, machine learning, and application development. The AWS Glue Crawler recognizes the partition structure of the dataset and populates the AWS Glue data catalog. Once crawled, the AWS Glue catalog groups data into logical tables and makes partition columns available for querying through Athena, which can be connected to any preferred business intelligence tool for visualization.</p> 
<p style="padding-left: 40px;">1. Search for <strong>AWS Glue</strong></p> 
<p style="padding-left: 40px;">2. Navigate to <strong>Crawlers</strong></p> 
<p style="padding-left: 40px;">3. Click on <strong>Add Crawler</strong></p> 
<p style="padding-left: 40px;">4. Create an <strong>AWS Glue table</strong>. See the screenshot below to get insights into the configuration used.</p> 
<p><img alt="" class="alignnone size-full wp-image-8286" height="772" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/23/Picture11-2.png" width="979" /></p> 
<p style="text-align: center;">&nbsp; &nbsp; <em>Crawler configuration</em></p> 
<h3>5) Query the data using Athena</h3> 
<p><strong>Daily Data View:</strong> In the below example we are creating a view from the sample data by flattening the json data.</p> 
<pre><code class="lang-sql">CREATE OR REPLACE VIEW view_name AS&lt;br /&gt;WITH&lt;br /&gt;dataset AS (&lt;br /&gt;SELECT&lt;br /&gt;schemaversion&lt;br /&gt;, nodeid&lt;br /&gt;, CAST("from_iso8601_timestamp"(messagetimestamp) AS timestamp) Message_TimeStamp&lt;br /&gt;, CAST("from_iso8601_timestamp"(messagetimestamp) AS date) Message_Date&lt;br /&gt;, subnodeid&lt;br /&gt;, compressed&lt;br /&gt;, "split_part"("replace"("replace"("replace"("json_format"(CAST(data AS json)), '"', ''), '[', ''), ']', ''), ',', 1) engine1_amp&lt;br /&gt;, CAST("split_part"("replace"("replace"("replace"("json_format"(CAST(data AS json)), '"', ''), '[', ''), ']', ''), ',', 2) AS double) value_engine1_amp&lt;br /&gt;, "split_part"("replace"("replace"("replace"("json_format"(CAST(data AS json)), '"', ''), '[', ''), ']', ''), ',', 3) engine1_vib&lt;br /&gt;, CAST("split_part"("replace"("replace"("replace"("json_format"(CAST(data AS json)), '"', ''), '[', ''), ']', ''), ',', 4) AS double) value_engine1_vib&lt;br /&gt;, "split_part"("replace"("replace"("replace"("json_format"(CAST(data AS json)), '"', ''), '[', ''), ']', ''), ',', 5) engine1_temp&lt;br /&gt;, CAST("split_part"("replace"("replace"("replace"("json_format"(CAST(data AS json)), '"', ''), '[', ''), ']', ''), ',', 6) AS double) value_engine1_temp&lt;br /&gt;FROM&lt;br /&gt;“athenaad“."assetbasedmonitoring"&lt;br /&gt;ORDER BY messagetimestamp DESC&lt;br /&gt;)&lt;br /&gt;SELECT *&lt;br /&gt;FROM&lt;br /&gt;dataset&lt;br /&gt;WHERE (Message_Date = current_date)&lt;br /&gt;ORDER BY Message_TimeStamp DESC</code></pre> 
<p><strong>Clean up the resources</strong></p> 
<p>To avoid incurring future charges, follow these steps to remove the example resources:</p> 
<p style="padding-left: 40px;">1. Delete the <strong>IoT Events model</strong>. Search IoT Events, Under Detector models select the model created and select <strong>Delete</strong>.</p> 
<p style="padding-left: 40px;">2. Delete the <strong>IoT Events Input</strong>. Search IoT Events, Under Inputs select the Input created and select <strong>Delete</strong>.</p> 
<p style="padding-left: 40px;">3. Delete the <strong>AWS Glue database</strong> and table. Search <strong>Glue</strong>, under Tables select the table that was created, click on <strong>Action</strong> drop down and select <strong>Delete table</strong>.</p> 
<p style="padding-left: 40px;">4. Delete <strong>AWS Athena</strong>. Search <strong>Athena</strong>, under <strong>Workgroups</strong> select the workgroup that was created, click on <strong>Action</strong> drop down and select <strong>Delete</strong>.</p> 
<h2>Conclusion</h2> 
<p>The configuration of AWS IoT Events and Amazon SNS helps to achieve faster response times during failure. In this post, we used a conveyor motor as an example, but the solution extends to a wide breadth of industries, such as products in a retail context.</p> 
<p>In a business context the solution reduces the downtime in a manufacturing or retail industry which sends out notifications, and improves data analytic capabilities for getting better insights.</p> 
<p>Please visit <a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/what-is-iotevents.html">AWS IoT Events</a> and <a href="https://docs.aws.amazon.com/sns/latest/dg/welcome.html">Amazon SNS</a> to learn more about configurations.</p> 
<h2>About the authors</h2> 
<table> 
 <tbody> 
  <tr> 
   <td> <p><img alt="" class="alignleft wp-image-8297 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/24/myphoto-150x150.png" width="150" /><strong>Prashanth Shankar Kumar</strong></p> <p>Prashanth Shankar Kumar is a Data Lake Data Architect at Amazon Web services. He specializes in building Big-data applications and help customer to modernize their applications on Cloud. He thinks Data is new oil and spends most of his time in deriving insights out of the Data.</p></td> 
  </tr> 
  <tr> 
   <td> <p><img alt="" class="alignleft wp-image-8307 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/06/24/vasanth-150x150.png" width="150" /><strong>Vasanth Jeyaraj</strong></p> <p>Vasanth Jeyaraj is a Cloud Infra Architect with Amazon Web Services (AWS). He supports enterprise customers in building well architected infrastructure with a focus on Automation, Infrastructure as Code, and also focussed on helping customers speed their cloud-native adoption journey by modernizing their platform infrastructure, internal architecture using MicroServices Strategy, Containerization, DevOps. Outside of work, he loves spending time with family and traveling.</p></td> 
  </tr> 
 </tbody> 
</table> 
<p>&nbsp;</p>
