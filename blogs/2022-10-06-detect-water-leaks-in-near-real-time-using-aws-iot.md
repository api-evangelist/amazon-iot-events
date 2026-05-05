---
title: "Detect water leaks in near real time using AWS IoT"
url: "https://aws.amazon.com/blogs/iot/detect-water-leaks-in-near-realtime-using-aws-iot/"
date: "Thu, 06 Oct 2022 21:28:30 +0000"
author: "Mrunal Daftari"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-events/feed/"
---
<h1>Introduction</h1> 
<p>Water is one of the most precious resources needed for the sustenance of life. However, only 2% of the global water supply is suitable for human consumption. The United States Environmental Protection Association (EPA) estimates that <a href="https://nepis.epa.gov/Exe/ZyNET.exe/60000I2A.txt?ZyActionD=ZyDocument&amp;Client=EPA&amp;Index=2006%20Thru%202010&amp;Docs=&amp;Query=&amp;Time=&amp;EndTime=&amp;SearchMethod=1&amp;TocRestrict=n&amp;Toc=&amp;TocEntry=&amp;QField=&amp;QFieldYear=&amp;QFieldMonth=&amp;QFieldDay=&amp;UseQField=&amp;IntQFieldOp=0&amp;ExtQFieldOp=0&amp;XmlQuery=&amp;File=D%3A%5CZYFILES%5CINDEX%20DATA%5C06THRU10%5CTXT%5C00000001%5C60000I2A.txt&amp;User=ANONYMOUS&amp;Password=anonymous&amp;SortMethod=h%7C-&amp;MaximumDocuments=1&amp;FuzzyDegree=0&amp;ImageQuality=r75g8/r75g8/x150y150g16/i425&amp;Display=hpfr&amp;DefSeekPage=x&amp;SearchBack=ZyActionL&amp;Back=ZyActionS&amp;BackDesc=Results%20page&amp;MaximumPages=1&amp;ZyEntry=2" rel="noopener noreferrer" target="_blank">1.7 trillion gallons</a>, roughly 30 percent of all treated water, is wasted every year in the United States.</p> 
<p>According to New York City Department of Environmental Protection, a leaking fire hydrant can waste up to 1,000 gallons of water per minute. Utilities have to deploy manual resources to identify these leaks, which can be a time consuming and labor-intensive process. Moreover, if these leaks are not addressed, then utilities can be penalized with heavy fines for non-compliance with environmental laws. These risks are an outcome of the water supply infrastructure seen in most cities, where supply lines are run underground, creating natural challenges to quickly identify and repair leaks. In most instances, leaks are never detected until the next scheduled maintenance call or during an emergency situation where use of fire hydrant is verified. A typical maintenance call can take up to 3 months and may require a minimum of 3 visits. Imagine how much water is wasted during this period. The solution described in this blog can help reduce leakage waste and maintenance costs for the utilities.</p> 
<h3>Solution overview</h3> 
<p>Consider a scenario, where water is flowing through fire hydrants across cities and rural areas, and somewhere along such long routes, a minor leak occurs. The leak remains undetected for several weeks to months. Even after the leak is detected, it could take a few more weeks to identify the exact location and then fix it. Today, most fire hydrants can be upgraded with an IoT sensor to communicate statistics on the status and usage of the fire hydrant. These sensors can help identify water leaks in almost real time to trigger proactive maintenance actions. In the proposed architecture, an IoT-enabled, 5G-capable fire hydrant communicates using MQTT protocols to AWS. These fire hydrants use <a href="https://aws.amazon.com/iot-core/" rel="noopener noreferrer" target="_blank">AWS IoT Core</a> to authenticate with AWS.</p> 
<p><img alt="Architecture diagram" class="aligncenter size-full wp-image-10648" height="512" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/Architecture-diagram.png" width="922" /></p> 
<h4 style="text-align: left;">Architecture diagram</h4> 
<p>Before we deep dive into the architecture, let’s understand the 4 stages represented by the architectural diagram.</p> 
<p>The first stage establishes communication from IoT devices to AWS over MQTT protocol.</p> 
<p>The second stage stores the collected data and maintains it for long-term compliance reasons.</p> 
<p>The third stage dispatches notifications to field teams for active leak situations and enables them to take actions ASAP.</p> 
<p>The fourth stage provides back-office operators control over the entire system. Back-office operators can utilize <a href="https://aws.amazon.com/quicksight/" rel="noopener noreferrer" target="_blank">Amazon QuickSight</a> dashboards to view active leaks and actions being performed, device health, and much more based on the collected data.</p> 
<p><strong>Stage 1: IoT Communications</strong></p> 
<p>5G-capable fire hydrants can establish communications with AWS over MQTT protocols.</p> 
<ul> 
 <li>AWS IoT Core helps establish communication. Bulk on-boarding of devices is possible with <a href="https://aws.amazon.com/iot-device-management/" rel="noopener noreferrer" target="_blank">AWS IoT Device Management</a>.</li> 
 <li>AWS IoT Device Management will securely access IoT devices, monitor health, detect and remotely troubleshoot problems, and manage software and firmware updates.</li> 
 <li><a href="https://aws.amazon.com/iot-device-defender/" rel="noopener noreferrer" target="_blank">AWS IoT Device Defender</a> helps maintain security of all on-boarded IoT devices, monitors security metrics, and generates alerts based on deviations from the expected behavior of each device.</li> 
 <li>Once this communication is established, the fire hydrant can send hydrant health status, geo-location and water flow data over secure MQTT protocol in JSON format to AWS.</li> 
</ul> 
<p><strong>Stage 2: Storage </strong></p> 
<p>Once IoT devices start sending events, there will be more data to collect and process.</p> 
<ul> 
 <li>Events are stored into <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> and then periodically offloaded to an archival service such as <a href="https://aws.amazon.com/s3/storage-classes/glacier/" rel="noopener noreferrer" target="_blank">Amazon S3 Glacier</a>.</li> 
</ul> 
<p><strong>Stage 3: Dispatch &amp; Fix</strong></p> 
<p>Events contain vital information about possible leaks. Now, connected fire hydrants can communicate with AWS so back-office teams can be notified in near real-time and alert field workers for immediate action.</p> 
<ul> 
 <li>Fire hydrant publishes a notification to AWS IoT Core over an MQTT protocol after a set interval.</li> 
 <li><a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-rules.html" rel="noopener noreferrer" target="_blank">AWS IoT Core Rules Engine</a> retrieves the notification from the MQTT topics.</li> 
 <li>AWS IoT Core Rules Engine then sends the notification to AWS IoT Events.</li> 
 <li><a href="https://aws.amazon.com/iot-events/" rel="noopener noreferrer" target="_blank">AWS IoT Events</a> has a detector model that monitors incoming IoT events, (e.g. a fire hydrant’s water flow and pressure level), by sending a request back to the AWS IoT Core MQTT topic.</li> 
 <li>AWS IoT Events detects the water flow and pressure abnormalities based on defined thresholds. If the water pressure and flow fall outside of defined thresholds, AWS IoT Events sends a notification message to an Amazon Simple Notification Service (Amazon SNS) topic.</li> 
 <li>The field operations team receives a notification message to inform field operators of the possible leak situation.</li> 
 <li>In addition to triggering an alert, AWS IoT Events sends the same message to <a href="https://aws.amazon.com/dynamodb/" rel="noopener noreferrer" target="_blank">Amazon DynamoDB</a> to create a support case. The back-office team tracks the status of fire hydrants using an Amazon QuickSight dashboard.</li> 
 <li>Once the leak is fixed, the field operations team updates the status of the support case.</li> 
</ul> 
<p><strong>Stage 4: Insight Reporting</strong></p> 
<p>All the collected events stored in Amazon S3 enable reporting capabilities.</p> 
<ul> 
 <li><a href="https://aws.amazon.com/athena/" rel="noopener noreferrer" target="_blank">Amazon Athena</a> is used to analyze and query on collected events.</li> 
 <li>Amazon QuickSight supports insightful dashboards for back-office operators to help them visualize active leaks, actions being performed for active leaks, geographical distribution of leaks, as well as help them monitor health status for devices and much more.</li> 
</ul> 
<h3>Prerequisites</h3> 
<p>To follow along and set up the asset inspection solution, be sure to have the following:</p> 
<ul> 
 <li>An AWS account.</li> 
 <li>A device or laptop/computer with an access to your AWS account, Python version 2.7.18+ installed, and the <a href="https://github.com/aws/aws-iot-device-sdk-python" rel="noopener noreferrer" target="_blank">AWS IoT Device SDK for Python</a> version 1.3.1+.</li> 
</ul> 
<h3>Setting up AWS IoT Events to manage fire hydrant leaks</h3> 
<p>An AWS IoT rule needs to be configured to forward device data from AWS IoT Core (MQTT topic) to AWS IoT Events.</p> 
<ul> 
 <li>Go to the AWS Management Console and select <strong>AWS IoT Core</strong>.</li> 
 <li>Select Message Routing, then Rules, and then choose <strong>Create rule</strong><em>.</em> Rule description is an optional field.</li> 
</ul> 
<p><img alt="IoT Rule set up" class="size-full wp-image-10649 aligncenter" height="318" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/IoT-Rule-set-up.png" width="860" /></p> 
<h4 style="text-align: left;">IoT Rule set up</h4> 
<ul> 
 <li>Set the <strong>Name</strong> for the rule and set the rule query statement to <em>SELECT * FROM ‘iot/topic</em>’. Sample query below.</li> 
</ul> 
<p><img alt="sample query" class="aligncenter size-full wp-image-10650" height="391" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/sample-query.png" width="860" /></p> 
<h4 style="text-align: left;">Sample Query</h4> 
<ul> 
 <li>Choose <strong>Add rule</strong><em> action</em>.</li> 
 <li>Select IoT Events option and enter <strong>Input name</strong>.</li> 
 <li>Select the <strong>Input</strong> previously created.</li> 
 <li>Select <strong>Create new role</strong> and enter a <strong>role name<em>.</em></strong></li> 
 <li>Select <strong>Add rule</strong><em> action</em>.</li> 
 <li>Select <strong>Create<em> rule</em></strong>.</li> 
 <li>A sample of the rule created in AWS Console.</li> 
</ul> 
<p><img alt="Create Rule" class="aligncenter size-full wp-image-10651" height="543" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/Create-Rule.png" width="725" /></p> 
<h4 style="text-align: left;">Create Rule</h4> 
<p>In AWS IoT Events, create the following components to start the fire hydrant’s water flow and pressure:</p> 
<ul> 
 <li>Select <strong>AWS IoT Events</strong> from the <strong>services</strong> menu. On the <strong>AWS IoT Events page</strong>, select an <strong>industry-specific template</strong> from the section shown following.</li> 
</ul> 
<p><img alt="Create Detector Model" class="aligncenter size-full wp-image-10652" height="408" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/Create-Detector-Model-intro.png" width="822" /></p> 
<h4 style="text-align: left;">Create Detector Model</h4> 
<ul> 
 <li>From that screen select <strong>Simple Alarm</strong> and choose <strong>Start</strong>.</li> 
</ul> 
<p><img alt="IoT Event template selection" class="aligncenter size-full wp-image-10653" height="454" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/IoT-Event-template-selection.png" width="908" /></p> 
<h4 style="text-align: left;">IoT Event template selection</h4> 
<ul> 
 <li>Create a <strong>detector model</strong> with 3 states each, with 2 transitions as shown following.</li> 
</ul> 
<p><img alt="Create Detector Model" class="aligncenter size-full wp-image-10654" height="429" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/Create-Detector-Model.png" width="968" /></p> 
<h4 style="text-align: left;">Create Detector Model</h4> 
<ul> 
 <li>Upon receipt of a notification, the device state is changed to “<strong>ActiveLeak</strong>“. This state is used to trigger the alert to field worker and back-office dashboard.</li> 
</ul> 
<h3>Creating an input in AWS IoT Events</h3> 
<p>You can create an input in AWS IoT Events by following the guide to <a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/iotevents-detector-input.html" rel="noopener noreferrer" target="_blank">create an input</a>. In our example, we create an input with the following details:</p> 
<ul> 
 <li>An example Input name set to “deviceNotificationInput”</li> 
 <li>Upload a JSON file with following example JSON payload:</li> 
 <li><code>{<br /> "geoLocation": "42.3928258265305, -71.07754968042828",<br /> "timeStamp": "2022-05-31 08:47:44.870092",<br /> "cityName": "Boston",<br /> "state": "MA",<br /> "deviceId": "BOS0”,<br /> "sensorHealth": "OK",<br /> "inputFlow": "10",<br /> "outputFlow": "10"<br /> }</code></li> 
</ul> 
<p><img alt="Create Device Notification Input" class="aligncenter size-full wp-image-10655" height="902" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/Create-Device-Notification-Input.png" width="968" /></p> 
<h4 style="text-align: left;">Create Device Notification Input</h4> 
<h3>Creating and publishing a detector model in AWS IoT Events</h3> 
<p>In our example, we create a detector model with the following details and you can find a sample at <a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/iotevents-examples-edwsaa.html" rel="noopener noreferrer" target="_blank">AWS IoT Events Developer Guide</a>.</p> 
<ul> 
 <li>Three states (Normal, ActiveLeak and Snooze).</li> 
 <li>Each with transitions that switches the device from one state to another.</li> 
 <li>Upon receiving a notification from the device, the normal transition triggers the sending of a notification to the outbound AWS IoT Core’s MQTT topic and changes the device state to out_of_range and pushing state to ActiveLeak.</li> 
 <li>If the repair is taking longer, then device state could be pushed to Snooze state while the fix is being performed.</li> 
</ul> 
<table> 
 <tbody> 
  <tr> 
   <td style="text-align: center;"><strong>Normal State</strong></td> 
   <td style="text-align: center;"><strong>ActiveLeak State</strong></td> 
   <td style="text-align: center;"><strong>Snooze State</strong></td> 
  </tr> 
  <tr> 
   <td><img alt="normal state" class="aligncenter size-full wp-image-10658" height="681" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/normal.png" width="300" /></td> 
   <td><img alt="active leak state" class="aligncenter size-full wp-image-10657" height="685" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/active-leak-state.png" width="310" /></td> 
   <td><img alt="snooze state" class="aligncenter size-full wp-image-10656" height="685" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/snooze-state.png" width="300" /></td> 
  </tr> 
 </tbody> 
</table> 
<h4 style="text-align: left;">States for the detector model</h4> 
<h3>Creating a detector model</h3> 
<ul> 
 <li>Create an IAM role for the Detector Model. For more information, see the documentation for <a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/iotevents-start.html#iotevents-permissions" rel="noopener noreferrer" target="_blank">setting up permissions for AWS IoT Events</a>.</li> 
 <li>Create the sample detector model as shown in the previous image with states and transitions.</li> 
</ul> 
<h3>Analyzing data using Amazon QuickSight</h3> 
<p>Following are the type of visualizations you can create with the data using <a href="https://docs.aws.amazon.com/quicksight/latest/user/creating-a-visual.html" rel="noopener noreferrer" target="_blank">Amazon QuickSight</a>.</p> 
<p><em><strong>Device health status</strong>: </em>The below chart shows the device status along with device locations of fire hydrants. These charts will assist back-office teams to identify faulty fire hydrants and send geolocation to field operators to fix those faulty devices.</p> 
<p><img alt="Device health report – Keeps back office up to date with compliance and device health status near real time" class="aligncenter size-full wp-image-10659" height="474" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/Device-health-report-–-Keeps-back-office-up-to-date-with-compliance-and-device-health-status-near-real-time-1.png" width="968" /></p> 
<h4 style="text-align: left;">Device health report – Keeps back office up to date with compliance and device health status near real time.</h4> 
<p><em><strong>Active leaks</strong>:</em> The below chart shows water leaks across 3 example cities – New York, Chicago, and Boston.</p> 
<p><img alt="Active leak report – Shows active leaks across cities for the back office" class="aligncenter size-full wp-image-10660" height="379" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/Active-leak-report-–-Shows-active-leaks-across-cities-for-the-back-office-1.png" width="968" /></p> 
<h4 style="text-align: left;">Active leak report – Shows active leaks across cities for the back office</h4> 
<p><em><strong>Geo-locations of active leaks</strong>:</em> The below chart shows active water leaks on a map, plotted using device Geo coordinates. This could be useful for back-office teams to actively look at streets where there is an issue.<br /> <img alt="geolocation of fire hydrants" class="aligncenter size-full wp-image-10616" height="562" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/05/geolocation-of-fire-hydrants.png" width="977" /></p> 
<h4 style="text-align: left;">Geo location of fire hydrants – Shows fire hydrants in map/street view with their status</h4> 
<h3>Cleaning up</h3> 
<p>To avoid incurring unwanted charges, delete the following resources:</p> 
<ul> 
 <li>Clean up IoT devices as explained <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-dc-cleanup.html" rel="noopener noreferrer" target="_blank">in this guide</a>.</li> 
 <li>Delete AWS Kinesis Delivery Streams as explained in this <a href="https://docs.aws.amazon.com/firehose/latest/APIReference/API_DeleteDeliveryStream.html" rel="noopener noreferrer" target="_blank">guide</a>. .</li> 
 <li>Delete Amazon S3 bucket as explained <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/delete-bucket.html" rel="noopener noreferrer" target="_blank">in this guide</a>.</li> 
 <li>Delete Amazon DynamoDB tables as explained <a href="https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SampleData.DeleteTables.html" rel="noopener noreferrer" target="_blank">in this guide</a>.</li> 
 <li>Delete Amazon Athena schema as explained <a href="https://docs.aws.amazon.com/athena/latest/ug/data-sources-managing.html" rel="noopener noreferrer" target="_blank">in this guide</a>.</li> 
 <li>Delete Amazon QuickSight charts/visuals as explained <a href="https://docs.aws.amazon.com/quicksight/latest/user/deleting-a-visual.html" rel="noopener noreferrer" target="_blank">in this guide</a>. Delete Dashboard explained in this <a href="https://docs.aws.amazon.com/quicksight/latest/user/deleting-a-dashboard.html" rel="noopener noreferrer" target="_blank">guide</a>, and delete account explained <a href="https://docs.aws.amazon.com/quicksight/latest/user/closing-account.html" rel="noopener noreferrer" target="_blank">in this guide</a>.</li> 
 <li>Delete Amazon SNS topics as explained <a href="https://docs.aws.amazon.com/sns/latest/dg/sns-delete-subscription-topic.html" rel="noopener noreferrer" target="_blank">in this guide</a>.</li> 
</ul> 
<h3>Conclusion</h3> 
<p>In this blog, we showed how AWS IoT services can be used to detect real-time leaks in fire hydrants and pin-point the exact location of the leaks. This solution can lead to a reduction in time to fix the leaks, thereby reducing water wastage and improving environmental impact. The same solution can be extended to detect leaks in home water appliances or leaks in oil/ natural gas pipelines.</p> 
<p>To learn more about how to use AWS IoT Core, you can refer to the <a href="https://aws.amazon.com/iot-core/" rel="noopener noreferrer" target="_blank">documentation</a>.</p> 
<p>AWS welcomes feedback. Please connect with us on LinkedIn if you have thoughts or questions.</p> 
<h3>Authors</h3> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="mrunal daftari" class="aligncenter size-full wp-image-10611" height="768" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/06/mrunal-new-photo.jpeg" width="576" /></td> 
   <td><strong>Mrunal Daftari</strong> is an Enterprise Senior Solutions Architect at Amazon Web Services. Mrunal is based in Boston, MA. He is a cloud enthusiast and very passionate about finding solutions for customers that are simple and addresses their business outcomes. He loves working with cloud technologies providing simple, scalable solutions that drive positive business outcomes, cloud adoption strategy, design innovative solutions and drive operational excellence.</td> 
  </tr> 
  <tr> 
   <td><img alt="" class="aligncenter size-full wp-image-10617" height="262" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/10/05/vaibhav.png" width="177" /></td> 
   <td><strong>Vaibhav Sabharwal</strong> is a Senior Solution Architect with Amazon Web Services (AWS). He is part of the Financial Service Technical Field Community. He helps AWS customers to build cloud adoption strategy, design innovative solutions and drive operational excellence.</td> 
  </tr> 
 </tbody> 
</table> 
<p>&nbsp;</p>
