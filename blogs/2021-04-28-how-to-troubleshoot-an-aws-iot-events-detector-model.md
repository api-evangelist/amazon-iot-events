---
title: "How to troubleshoot an AWS IoT Events detector model"
url: "https://aws.amazon.com/blogs/iot/troubleshoot-aws-iot-events-detector-model/"
date: "Wed, 28 Apr 2021 20:01:47 +0000"
author: "Vaibhav Sharma"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-events/feed/"
---
<p><a href="https://aws.amazon.com/iot-events/">AWS IoT Events</a> recently launched a <a href="https://aws.amazon.com/about-aws/whats-new/2021/02/new-troubleshooting-feature-generally-available-aws-iot-events/">new troubleshooting feature</a> that automatically analyzes your detector model for potential syntax errors, structural issues, and runtime errors without needing to publish the detector model first. In this post, you will learn how to use this new feature with your AWS IoT Events detector model.</p> 
<p>AWS IoT Events is a fully managed service that makes it easy to detect and respond to events from IoT sensors and applications.&nbsp;The detector model in AWS IoT Events lets you monitor your equipment or device fleets for failures or changes in operation and trigger actions when such events occur. To learn more about detector models, see the&nbsp;<a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/iotevents-getting-started.html">Getting Started with the AWS IoT Events Console</a>&nbsp;guide.</p> 
<p>Prior to the launch of this new feature, to check if a detector model works as expected, customers would first send sample data to the detector model. Next, they would execute the <a href="https://docs.aws.amazon.com/iotevents/latest/apireference/API_iotevents-data_DescribeDetector.html">DescribeDetector</a>&nbsp;API to check if the detector state changed as expected. In the case where the detector’s state did not have the expected change, they would then identify the root cause, publish an updated version of the detector model, and then send the sample data again to the detector model. They would keep repeating this debugging cycle until their&nbsp;detector model achieved their desired functionality (or they ran out of time). Such debugging can be time consuming,&nbsp;especially for a complex detector model.</p> 
<p>Here is an example of how debugging can be time consuming. Our detector model continuously monitors the temperature of a room and turns on the heating or cooling mode for the HVAC system as needed to maintain temperature between 68-72 degrees Fahrenheit. The detector computes and stores the average temperature in a variable named <code>averageTemperature</code>.</p> 
<p><img alt="Diagram of HVAC detector model in AWS IoT Events console" class="alignnone size-full wp-image-4902" height="774" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/27/HVAC-detector-model-in-AWS-IoT-Events-console-1.png" width="2540" /></p> 
<p>After sending sample data to this example detector model, we observe that the average temperature is not recomputed. Checking the logs in <a href="https://docs.aws.amazon.com/iotevents/latest/developerguide/monitoring-cloudwatch.html">Amazon CloudWatch</a> for the detector model reveals no information as to why the average temperature is not recomputed. We check the following condition of the detector model that needs to be satisfied in order to recompute the average temperature.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">($variable.resetMe == true) &amp;&amp;&nbsp;($input.temperatureInput.sensorData.temperature &lt; 80 &amp;&amp;&nbsp;$input.temperatureInput.sensorData.temperature &gt; 60)</code></pre> 
</div> 
<p>We have set the variable&nbsp;<code>resetMe</code>&nbsp;to <code>"true"</code>, a value of String&nbsp;data type, when the detector enters the <strong>start</strong>&nbsp;state.&nbsp;After several attempts, we realize that the value, <code>"true"</code>&nbsp;(note the quotation marks) can never equal&nbsp;<code>true</code>, a Boolean value. This causes the condition&nbsp;<code>($variable.resetMe == true)</code>&nbsp;to always evaluate to&nbsp;false.</p> 
<p>This&nbsp;<a href="https://aws.amazon.com/about-aws/whats-new/2021/02/new-troubleshooting-feature-generally-available-aws-iot-events/">new troubleshooting feature</a> of AWS IoT Events catches this mismatch of data types much earlier by flagging a warning for the <code>resetMe</code> variable. Without the feature, debugging a much bigger detector model with 20 states and 50 variables, as an example, can be a time-consuming exercise. This new feature performs seven different analyses on your detector model for potential syntax errors (e.g. bad expressions or payloads), structural issues (e.g. missing states or input triggers) and runtime errors (e.g. data type mismatch, missing data, potential to hit service limits, etc.) before publishing the model.</p> 
<h2>To get&nbsp;started with troubleshooting your detector model</h2> 
<p>This step-by-step walkthrough consists of the following sections to debug the previous example detector model:</p> 
<ol> 
 <li>Prerequisites</li> 
 <li>Creating an input</li> 
 <li>Creating a detector model</li> 
 <li>Troubleshooting issues with the detector model</li> 
</ol> 
<h3>Prerequisites</h3> 
<p>For this use case, make sure that you have an AWS account in the same AWS Region where AWS IoT Events is available. This use case uses the US East (N. Virginia) Region. However, you can choose another <a href="https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/">AWS Region</a> where AWS IoT Events is available.</p> 
<h3>To create an input for the detector model</h3> 
<p>Create an input named&nbsp;<strong>temperatureInput</strong> by following the steps:</p> 
<ol> 
 <li>Save the following JSON in a file on your computer. 
  <div class="hide-language"> 
   <pre><code class="lang-json">{
  "sensorId": 1,
  "areaId": 1,
  "sensorData" : {
    "temperature": 70
  }
}</code></pre> 
  </div> </li> 
 <li>Navigate to the <a href="https://console.aws.amazon.com/iotevents/">AWS IoT Events console</a>, then choose&nbsp;<strong>Inputs</strong>, <strong>Create input.</strong></li> 
 <li>For&nbsp;<strong>Input name</strong>, enter&nbsp;“temperatureInput”.</li> 
 <li>Choose&nbsp;<strong>Upload a JSON file.</strong></li> 
 <li>In the dialog box, choose the JSON file that you created in step 1.</li> 
 <li>Choose&nbsp;<strong>Create</strong> to create the input.</li> 
 <li>Once the input is created successfully, you will be redirected to the AWS IoT Events inputs console page with the following message.</li> 
</ol> 
<p><img alt="Success flashbar that indicates you have successfully created an input" class="alignnone size-full wp-image-4901" height="144" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/27/Input-created-succesfully-in-AWS-IoT-Events-console-1.png" width="1716" /></p> 
<h3>To create a detector model</h3> 
<p>Create a draft detector model named <strong>areaDetectorModel</strong> by following these steps:</p> 
<ol> 
 <li>Download the <a href="https://github.com/aws-samples/aws-troubleshoot-iotevents-detector-model/blob/main/sample-detector-model.json">JSON file</a>.</li> 
 <li>Navigate to the <a href="https://console.aws.amazon.com/iotevents/">AWS IoT Events console</a>, then choose&nbsp;<strong>Detector models.</strong></li> 
 <li>Choose&nbsp;<strong>Create detector model</strong> to navigate to <strong>Create your detector model</strong> console page.</li> 
 <li>Choose <strong>Import detector model </strong>from the left pane and then choose&nbsp;<strong>Import to </strong>select&nbsp;the JSON file for the detector model that you downloaded in step 1.</li> 
 <li>Once the detector model is created successfully, you will be redirected to the canvas:</li> 
</ol> 
<p><img alt="Success flashbar that indicates you have successfully imported your detector model with the diagram of the HVAC detector model" class="alignnone size-full wp-image-4900" height="514" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/27/Detector-model-created-succesfully-in-AWS-IoT-Events-console-1.png" width="1296" /></p> 
<h3>To troubleshoot issues with the detector model</h3> 
<p>Troubleshoot the detector model by&nbsp;following these steps:</p> 
<ol> 
 <li>Choose&nbsp;<strong>Run analysis</strong>.<img alt="Figure highlighting the &quot;Run analysis&quot; button in the AWS IoT Events console" class="alignnone size-full wp-image-4899" height="112" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/27/Run-Analysis-button-in-AWS-IoT-Events-console-1.png" width="2145" /></li> 
 <li>Wait until the analysis completes and results are displayed in the&nbsp;<strong>Detector model analysis</strong> panel. Choose <strong>Warning</strong> to see the details for the incompatible data types:</li> 
</ol> 
<p><img alt="Figure showing the results from analyzing the detector model in the AWS IoT Events console. The results include a warning for data types used in your detector model. The warning message is &quot;Incompatible data types [Boolean, String] used with $variable.resetMe. This may lead to a runtime error.&quot;" class="alignnone size-full wp-image-4898" height="241" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/27/HVAC-detector-model-analysis-warning-from-AWS-IoT-Events-console-1.png" width="1301" /></p> 
<ol start="3"> 
 <li>To fix this warning, choose&nbsp;<strong>Start </strong>state in the imported detector model.</li> 
 <li>In the <strong>State </strong>pane, choose <strong>prepare</strong> event under&nbsp;<strong>onEnter</strong>.</li> 
 <li>On the <strong>Add OnEnter event </strong>page, expand the first <strong>Set variable</strong> action.</li> 
 <li>Change the <strong>Assign value </strong>of <strong>resetMe</strong>&nbsp;variable from the String&nbsp;value of&nbsp;<strong>“true”</strong> to the Boolean value of&nbsp;<strong>true</strong>.</li> 
</ol> 
<p><img alt="Figure showing a variable resetMe being assigned to the Boolean value, true, in the AWS IoT Events console" class="alignnone size-full wp-image-4894" height="448" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/27/Changing-resetMe-value-to-true-in-AWS-IoT-Events-console-1.png" width="982" /></p> 
<ol start="7"> 
 <li>Scroll down the list of all the actions and choose&nbsp;<strong>Save</strong>.</li> 
 <li>Choose <strong>Rerun analysis.</strong></li> 
</ol> 
<p><img alt="Figure highlighting the &quot;Rerun analysis&quot; button in the AWS IoT Events console" class="alignnone size-full wp-image-4904" height="79" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/27/Rerun-analysis-in-AWS-IoT-Events-console-1.png" width="1672" /></p> 
<ol start="9"> 
 <li>Wait until the analysis completes and results are displayed in the&nbsp;<strong>Detector model analysis </strong>panel with&nbsp;no errors and no warnings.</li> 
</ol> 
<p><img alt=" Figure showing results from re-running analysis on the detector model in the AWS IoT Events console" class="alignnone size-full wp-image-4893" height="154" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/27/Analysis-on-HVAC-detector-model-with-no-errors-and-warnings-in-AWS-IoT-Events-console-1.png" width="1309" /></p> 
<ol start="10"> 
 <li>You can also scroll down the list in the&nbsp;<strong>Detector model analysis </strong>panel to validate that the <code>resetMe</code> variable now has the Boolean data type as shown:</li> 
</ol> 
<p><img alt="An informative message from the AWS IoT Events console indicating &quot;Inferred data types [Boolean] for $variable.resetMe&quot;" class="aligncenter wp-image-4876 size-full" height="87" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/27/Boolean-data-type-inferred-for-resetMe-variable-in-AWS-IoT-Events-console.png" width="629" /></p> 
<p>Now your detector model is ready to be published!</p> 
<h2>Conclusion</h2> 
<p>In this post, we demonstrated how to use the recently launched&nbsp;<a href="https://aws.amazon.com/about-aws/whats-new/2021/02/new-troubleshooting-feature-generally-available-aws-iot-events/">new troubleshooting feature</a>&nbsp;for AWS IoT Events that automatically analyzes your detector model for potential syntax errors, structural issues, and runtime errors without needing to publishing the detector model first.</p> 
<p>Now that detector models can be analyzed for errors before publishing, explore using them for the following use cases:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/blogs/iot/monitor-iot-device-geolocation-with-aws-iot-events/">Monitor IoT device geolocation</a></li> 
 <li><a href="https://aws.amazon.com/blogs/iot/collecting-organizing-monitoring-and-analyzing-industrial-data-at-scale-using-aws-iot-sitewise-part-3/">Enable conditions monitoring and send notifications or alerts in near real-time for industrial data at scale&nbsp;</a></li> 
 <li><a href="https://aws.amazon.com/blogs/iot/asset-maintenance-with-aws-iot-services-predict-and-respond-to-potential-failures-before-they-impact-your-business/">Predict and respond to potential failures before they impact your business</a></li> 
</ul> 
<h2>About the authors</h2> 
<table> 
 <tbody> 
  <tr> 
   <td><img alt="Author image of Vaibhav Sharma" class="alignnone size-full wp-image-4931" height="400" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/28/bio-photo-Vaibhav-Sharma-size-400.jpeg" width="300" /></td> 
   <td><strong>Vaibhav Sharma</strong> is an Applied Scientist in the Automated Reasoning Group. He works on applying automated reasoning to solve customer problems in the IoT domain.</td> 
  </tr> 
  <tr> 
   <td><img alt="Author Image of Andrew Apicelli" class="alignnone size-full wp-image-4932" height="400" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/04/28/bio-photo-Andrew-Apicelli-size-400.jpeg" width="300" /></td> 
   <td><strong>Andrew Apicelli</strong> is a Software Development Engineer at Amazon Web Services IoT. He focuses on developing AWS IoT cloud services that help customers across many industries achieve positive business outcomes.</td> 
  </tr> 
 </tbody> 
</table> 
<p>&nbsp;</p>
