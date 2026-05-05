---
title: "Improve your security posture with AWS IoT Device Defender direct integration with AWS Security Hub"
url: "https://aws.amazon.com/blogs/iot/improve-your-security-posture-with-aws-iot-device-defender-direct-integration-with-aws-security-hub/"
date: "Fri, 18 Nov 2022 00:49:06 +0000"
author: "Ryan Dsouza"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-device-defender/feed/"
---
<h2>Introduction</h2> 
<p>We are excited to announce that <a href="https://aws.amazon.com/iot-device-defender/">AWS IoT Device Defender</a> is now integrated with <a href="https://aws.amazon.com/security-hub/?nc2=h_ql_prod_se_sh">AWS Security Hub</a>. This integration allows you to ingest alarms and their attributes from audit and detect features in one central location, without custom coding. This will help you offload or reduce complexity of managing disparate workflows from multiple security consoles when you review devices monitored by AWS IoT Device Defender.</p> 
<p>You can use AWS IoT Device Defender to audit and monitor your IoT devices and can use AWS Security Hub to centralize and prioritize security findings from across AWS accounts, services, and supported third-party partners to help analyze security trends and identify the highest priority security issues. With the direct integration of AWS IoT Device Defender to AWS Security Hub, you can view AWS IoT Device Defender alarms alongside events from other AWS security services to centrally view and improve the security posture of your IoT solution.</p> 
<p>AWS Security Hub ingests findings from multiple AWS services, including <a href="https://aws.amazon.com/guardduty/">Amazon GuardDuty</a>, <a href="https://aws.amazon.com/inspector/">Amazon Inspector</a>, <a href="https://aws.amazon.com/macie/">Amazon Macie</a>, <a href="https://aws.amazon.com/firewall-manager/?nc2=h_m1">AWS Firewall Manager</a>, <a href="https://aws.amazon.com/iam/">AWS Identity and Access Management</a> (IAM) Access Analyzer, and <a href="https://aws.amazon.com/systems-manager/">AWS Systems Manager</a> Patch Manager. With the AWS AWS IoT Device Defender integration to AWS Security Hub, you can ingest AWS IoT Device Defender alarms into AWS Security Hub. Findings from each service are normalized into the <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-findings-format.html">AWS Security Finding Format</a> (ASFF), so that you can review findings in a standardized format and take action quickly. You can use AWS Security Hub to provide a centralized view of all security-related findings, where you can set up alerting and automatic remediation.</p> 
<h3>Solution overview</h3> 
<p><img alt="" class="alignnone size-full wp-image-11198" height="175" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/11/16/architecture1.png" width="896" /></p> 
<p><strong>Figure 1:</strong> Solution architecture</p> 
<h3>Prerequisites</h3> 
<ul> 
 <li>You must have AWS Security Hub set up in the Region where you’re deploying the solution. To set up, refer to the <a href="https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-settingup.html">Setting up AWS Security Hub</a> documentation page.</li> 
 <li>AWS IoT Core Console <a href="https://docs.aws.amazon.com/iot/latest/developerguide/view-mqtt-messages.html">MQTT test client</a> access.</li> 
 <li>Note that for <a href="https://docs.aws.amazon.com/iot/latest/developerguide/detect-device-side-metrics.html">device-side metrics</a> and <a href="https://docs.aws.amazon.com/iot/latest/developerguide/dd-detect-custom-metrics.html">custom metrics</a>, you will need to setup a device side agent with our <a href="https://github.com/aws-samples/aws-iot-device-defender-agent-sdk-python">sample agent</a>&nbsp;in Python or use <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-sdks.html">AWS IoT Device SDK</a>.</li> 
</ul> 
<h3>Solution walk-through</h3> 
<p>AWS Security Hub integrations allow aggregating security finding data from several AWS services and from supported <a href="https://aws.amazon.com/security-hub/partners/">AWS Partner Network (APN) security solutions</a>. The Integrations page in the AWS Security Hub console provides access to all of the available AWS and third-party product integrations. The <a href="https://docs.aws.amazon.com/securityhub/1.0/APIReference/API_Operations.html">AWS Security Hub API</a> also provides operations to allow you to manage integrations.</p> 
<p><img alt="" class="alignnone size-full wp-image-11197" height="954" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/11/16/figure2.png" width="1429" /></p> 
<p><strong>Figure 2:</strong> AWS Security Hub console showing AWS IoT Device Defender integrations</p> 
<p>Navigate to <a href="https://us-east-1.console.aws.amazon.com/securityhub/home?region=us-east-1#/integrations">AWS IoT Security Hub &gt; Integrations</a> page to see and accept findings from AWS IoT Device Defender service for your use case.</p> 
<ol> 
 <li>Under <strong>Integrations</strong> section, filter for integrations, enter <strong>Device Defender</strong>.</li> 
 <li>Choose <strong>Accept findings</strong> for both audit and detect integrations.</li> 
</ol> 
<p>Congratulations! You have enabled accepting AWS IoT Device Defender audit and detect findings to AWS Security Hub. You can continue with following experiment sections to try and test integrations in your AWS account.</p> 
<p><strong>Experimenting AWS IoT Device Defender audit findings integration with AWS Security Hub</strong></p> 
<p>An AWS IoT Device Defender audit looks at account and device related settings and policies to ensure security measures are in place. To experiment an audit finding, you can create an overly permissive device policy and run the audit on demand to be able to generate findings right away.</p> 
<ol> 
 <li>Navigate to <a href="https://us-east-1.console.aws.amazon.com/iot/home?region=us-east-1#/policyhub">AWS IoT &gt; Security &gt; Policies</a>.</li> 
 <li>Choose Create Policy</li> 
 <li>Under Policy properties section, for Policy name, specify a name for the policy.</li> 
 <li>Under the Policy document, prepare an overly permissive statement using the following: 
  <ul> 
   <li>For Policy effect, choose <strong>Allow</strong></li> 
   <li>For Policy action, choose * (all AWS IoT Actions)</li> 
   <li>For Policy resource, enter * (corresponds to all AWS IoT resources)</li> 
   <li>Choose <strong>Create.</strong></li> 
  </ul> </li> 
</ol> 
<p>Now you’ve created an overly permissive device policy in your AWS account. It will be detected as a security finding with critical severity for the next AWS IoT Device Defender Audit run. You can run an on-demand audit to see the results right away.</p> 
<ol> 
 <li>Navigate to <a href="https://us-east-1.console.aws.amazon.com/iot/home?region=us-east-1#/dd/scheduledAuditsHub">AWS IoT &gt; Security &gt; Audit &gt; Schedules</a>.</li> 
 <li>Under Scheduled audits, choose Create.</li> 
 <li>On the following page, under Available checks, select all checks.</li> 
 <li>Under Set schedule, for Recurrence, choose <strong>Run audit now (once).</strong></li> 
</ol> 
<p>The audit is started and will turn from in-progress to not compliant within a few minutes. Choose the latest audit, on the audit Report page, review the <strong>Non-compliant checks</strong> section.</p> 
<p><img alt="" class="alignnone size-full wp-image-11196" height="917" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/11/16/figure3.png" width="1430" /></p> 
<p><strong>Figure 3:</strong> AWS IoT Device Defender audit report</p> 
<p>Your recently created overly permissive IoT policy is detected by the AWS IoT Device Defender audit. Now you can navigate to AWS Security Hub console to check the findings reported by AWS IoT Device Defender audit.</p> 
<ol> 
 <li>Navigate to <a href="https://us-east-1.console.aws.amazon.com/securityhub/home?region=us-east-1#/integrations">AWS IoT Security Hub &gt; Integrations</a> page.</li> 
 <li>Under Integrations section, for filter integrations, enter <strong>Device Defender</strong>.</li> 
 <li>Under <strong>AWS IoT Device Defender – Audit</strong>, choose <strong>See findings</strong>.</li> 
</ol> 
<p><img alt="" class="alignnone size-full wp-image-11195" height="405" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/11/16/figure4.png" width="1430" /></p> 
<p><strong>Figure 4:</strong> AWS IoT Device Defender audit findings in AWS Security Hub</p> 
<p>Congratulations! You have integrated AWS Security Hub with AWS IoT Device Defender audit findings. Findings in AWS Security Hub are identified by the audit check type as the title and the checked resource identifier. In this example, you will notice “AwsIotPolicy” and “AwsIotAccountSettings” were the non-compliant resource types. Also, audit sends check summaries to AWS Security Hub, which include status, number of resources checked, percentage of non-compliance about an audit task for each check type. The summaries can be identified by its’ title or resource type “AwsIotAuditTask”. You can click each finding and check finding details and trigger workflow actions.</p> 
<p><img alt="" class="alignnone size-full wp-image-11194" height="716" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/11/16/figure5-1.png" width="745" /></p> 
<p><strong>Figure 5:</strong> AWS IoT Device Defender audit finding details in AWS Security Hub</p> 
<p>You can continue to the following section to also experiment detect findings.</p> 
<p><strong>Experimenting AWS IoT Device Defender Detect findings integration with AWS Security Hub</strong></p> 
<p>With AWS IoT Device Defender Detect, you can identify unusual behavior that might indicate a compromised device by monitoring the behavior of your devices. You create security profiles, which contain definitions of expected device behaviors, and assign them to a group of devices or to all the devices in your fleet. To experiment with a detect finding, you can create a security profile with a simple expected AWS IoT Core thing behavior, and then connect using an IoT device client that conflicts with the expected behavior.</p> 
<ol> 
 <li>Navigate to the Security Profiles section of the AWS IoT Device Defender Console: <a href="https://us-east-1.console.aws.amazon.com/iot/home?region=us-east-1#/dd/securityProfilesHub">AWS IoT &gt; Manage &gt; Security &gt; Detect &gt; Security Profiles</a></li> 
 <li>Choose <strong>Create Security Profile</strong> and choose <strong>Create Rule-based anomaly detect profile</strong></li> 
 <li>For Target, choose <strong>All things</strong></li> 
 <li>Specify a Security Profile name</li> 
 <li>Clear all Cloud-side metrics, except <strong>Message size</strong></li> 
 <li>Choose <strong>Next</strong></li> 
 <li>Under the Define metric behaviors section, specify the following parameters for <strong>Message size</strong>: 
  <ul> 
   <li>Check type: <strong>Absolute</strong></li> 
   <li>Operator: <strong>Less than</strong></li> 
   <li>Value: <strong>8</strong></li> 
  </ul> </li> 
 <li>Keep the others as default, and Choose <strong>Next.</strong></li> 
 <li>Choose <strong>Create.</strong></li> 
</ol> 
<p>This defines a device behavior that expected message size is less than 8 bytes.</p> 
<p>Now, use your IoT devices with <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-sdks.html">AWS IoT device client/SDKs</a> or AWS IoT Core Console <a href="https://us-east-1.console.aws.amazon.com/iot/home?region=us-east-1#/test">MQTT test client</a> to publish messages bigger than 8 bytes on average.</p> 
<p>Within five minutes time frame, an AWS IoT Device Defender detect finding will be produced. Navigate to <a href="https://us-east-1.console.aws.amazon.com/iot/home?region=us-east-1#/dd/violationhub">AWS IoT &gt; Security &gt; Detect &gt; Alarms</a> and view produced findings under <strong>All alarms.</strong></p> 
<p>Now you can navigate to the AWS Security Hub console to view the findings reported by AWS IoT Device Defender Detect.</p> 
<ol> 
 <li>Navigate to <a href="https://us-east-1.console.aws.amazon.com/securityhub/home?region=us-east-1#/integrations">AWS IoT Security Hub &gt; Integrations</a> page.</li> 
 <li>Under Integrations section, for filter integrations, enter <strong>Device Defender.</strong></li> 
 <li>Under <strong>AWS IoT Device Defender – Detect</strong>, choose <strong>See findings.</strong></li> 
</ol> 
<p><img alt="" class="alignnone size-full wp-image-11193" height="325" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/11/16/figure6-1.png" width="1645" /></p> 
<p><strong>Figure 6:</strong> AWS IoT Device Defender Detect findings in AWS Security Hub</p> 
<p>Congratulations! You have integrated AWS Security Hub with AWS IoT Device Defender Detect findings. You will notice that findings for violations are sent to AWS Security Hub in near real time. Violations can be identified by their Thing name and Behavior name in the Title and time that the violations are detected. After a violation goes out of alarm, the corresponding Security Hub finding is immediately archived. You can click each finding and check finding details and trigger workflow actions.</p> 
<p><img alt="" class="alignnone size-full wp-image-11192" height="699" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/11/16/figure7.png" width="662" /></p> 
<p><strong>Figure 7:</strong> AWS IoT Device Defender Detect finding details in AWS Security Hub</p> 
<p>Note that, you can also use AWS IoT Device Defender <a href="https://docs.aws.amazon.com/iot/latest/developerguide/dd-detect-ml.html">ML Detect</a> to set the normal device behavior. AWS IoT Device Defender then identifies anomalies and triggers alarms using the Machine Learning (ML) models. These alarms are sent to AWS Security Hub and can be seen in the AWS Security Hub console as described earlier.</p> 
<h3>Conclusion</h3> 
<p>In this post, you’ve learned how to set up <a href="https://aws.amazon.com/iot-device-defender/">AWS IoT Device Defender</a> to send audit and detect findings to&nbsp;<a href="https://aws.amazon.com/security-hub/?p=ft&amp;c=sc&amp;z=3">AWS Security Hub</a> to gain a centralized view of security findings across the services running on the cloud and the edge. By ingesting security events into AWS, you can triage alarms and get, deeper insights and situational awareness of your IoT and cloud security posture. The solution can be extended using additional AWS services, including <a href="https://aws.amazon.com/eventbridge/">Amazon EventBridge</a>, <a href="https://aws.amazon.com/lambda/">AWS Lambda</a>, and <a href="https://aws.amazon.com/dynamodb/?trk=ea446940-00bb-4bee-9f27-d7a9a8080e4d&amp;sc_channel=ps&amp;sc_campaign=acquisition&amp;sc_medium=ACQ-P%7CPS-GO%7CBrand%7CDesktop%7CSU%7CDatabase%7CDynamoDB%7CUS%7CEN%7CText&amp;s_kwcid=AL!4422!3!536393513269!e!!g!!amazon">Amazon DynamoDB</a> to correlate AWS Security Hub findings from multiple AWS security services. To learn more, read <a href="https://aws.amazon.com/blogs/security/correlate-security-findings-with-aws-security-hub-and-amazon-eventbridge/">correlate security findings with AWS Security Hub and Amazon EventBridge</a>. You can also reference <a href="https://www.youtube.com/watch?v=oZnt2TK17p8">this video</a> for a live demo of the solution.</p> 
<h2>Authors</h2> 
<table> 
 <tbody> 
  <tr> 
   <td><a href="https://www.linkedin.com/in/ryandsouzaaws/"><span style="color: #3366ff;"><span style="color: #333399;"><strong>Ryan Ds</strong><strong>ouz</strong></span><strong><span style="color: #333399;">a</span></strong> </span></a>is a Principal Solutions Architect for IoT at AWS. Based in New York City, Ryan helps customers design, develop, and operate more secure, scalable, and innovative solutions using the breadth and depth of AWS capabilities to deliver measurable business outcomes. Ryan has over 25 years of experience in digital platforms, smart manufacturing, energy management, building and industrial automation, and OT/IIoT security across a diverse range of industries. Before AWS, Ryan worked for Accenture, SIEMENS, General Electric, IBM, and AECOM, serving customers for their digital transformation initiatives.</td> 
  </tr> 
  <tr> 
   <td><a href="https://www.linkedin.com/in/josephseungchoi/"><strong>Joseph Choi</strong> </a>is a Sr. Product Manager-Tech at AWS IoT. He focuses on building services that help device makers, automotive manufacturers, IoT providers monitor and secure their devices.</td> 
  </tr> 
  <tr> 
   <td><a href="https://www.linkedin.com/in/eercanayar/"><strong>Emir Ayar</strong></a> is a Tech Lead Solutions Architect on the AWS Prototyping team. He specializes in helping customers build IoT, ML at the Edge, and Industry 4.0 solutions and implement architectural best practices. He lives in Luxembourg and enjoys playing synthesizers.</td> 
  </tr> 
 </tbody> 
</table> 
<p></p>
