---
title: "Identify misconfigured IoT policies using AWS IoT Device Defender"
url: "https://aws.amazon.com/blogs/iot/identify-misconfigured-iot-policies-using-aws-iot-device-defender/"
date: "Fri, 10 Feb 2023 16:37:59 +0000"
author: "Ryan Dsouza"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-device-defender/feed/"
---
<h2>Introduction</h2> 
<p>We are excited to announce a new <a href="https://aws.amazon.com/about-aws/whats-new/2022/12/aws-iot-device-defender-audit-identifies-potential-misconfiguration-iot-policies/">AWS IoT Device Defender audit feature</a> to identify potential misconfigurations when using wild cards in Internet of Things (IoT) policies. <a href="https://aws.amazon.com/iot-device-defender/">AWS IoT Device Defender</a> is a fully managed IoT security service that enables you to audit and monitor your IoT device fleet and secure your IoT configurations on an ongoing basis. Security misconfigurations, such as overly permissive policies that permit a device access to unintended resources, can be a major cause of security incidents and compromise the security posture of your solution. With the new AWS IoT Device Defender <a href="https://docs.aws.amazon.com/iot/latest/developerguide/audit-chk-iot-misconfigured-policies.html">IoT policy potentially misconfigured</a> audit feature, you can more easily identify flaws, troubleshoot issues, and take necessary corrective actions. This can help you improve the security posture of your IoT solutions.</p> 
<h2>Background</h2> 
<p><a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-policies.html">AWS IoT Core policies</a> are JSON documents. They follow the same conventions as AWS Identity and Access Management (IAM) policies. With AWS IoT Core policies, you can control access to the AWS IoT Core data plane. The AWS IoT Core data plane consists of operations that enable you to connect to the AWS IoT Core message broker and send and receive MQTT messages. Similarly, data plane operations can also help you get or update the state of your device through <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html">AWS IoT Device Shadow</a>, a feature of AWS IoT Core that makes a device’s state available to apps and other services, whether the device is connected to AWS IoT or not.</p> 
<p>In some cases, customers might misconfigure IoT policies because of confusion between IoT policy wildcards and MQTT wildcards. If a customer configures IoT policies in a certain way, it is possible to over-subscribe devices to receive data on topics when the devices should have been explicitly denied subscription.</p> 
<p>In this blog, we discuss two types misconfigurations and how you can use AWS IoT Device Defender audit to identify and fix these potential misconfigurations in IoT policies.</p> 
<h2>Using wildcard characters in MQTT and AWS IoT Core policies</h2> 
<p>MQTT and AWS IoT Core policies have different wildcard characters and you should choose them after careful consideration. In MQTT, the wildcard characters ‘+’ and ‘#’ are used in MQTT topic filters to subscribe to multiple topic names. Character ‘+’ for single MQTT topic level, and ‘#’ for multiple MQTT topic levels. AWS IoT Core policies use ‘*’ and ‘?’ as wildcard characters and follow the conventions of IAM policies. In a policy document, the ‘*’ represents any combination of characters and a question mark ‘?’ represents any single character. In policy documents, the MQTT wildcard characters, ‘+’ and ‘#’ are treated as those characters having no special meaning. To describe multiple topic names and topic filters in the resource attribute of a policy, use the ‘*’ and ‘?’ wildcard characters in place of the MQTT wildcard characters.</p> 
<p>When choosing wildcard characters to use in a policy document, consider that the ‘*’ character is not confined to a single topic level as the ‘+’ character is in an MQTT topic filter. To help constrain a wildcard specification to a single MQTT topic filter level, consider using multiple ‘?’ characters.&nbsp;Refer to the documentation <a href="https://docs.aws.amazon.com/iot/latest/developerguide/pub-sub-policy.html#pub-sub-policy-cert">for examples</a> of wildcard characters used in MQTT and AWS IoT Core policies for MQTT clients.</p> 
<p>There are 2 types of misconfigurations:</p> 
<p><strong>Type 1:</strong> When customers want a device to receive messages for a whole topic space ‘<em>building/*</em>’ but not for specific sub-topics related to ‘<em>building/control_room/*</em>’.</p> 
<p>In this example, topic filters are intended to deny access, but the use of wildcard results in allowing access. In a policy that contains topic filter with wildcards in allow statements, and a deny statement that has a subset of allow resources, the deny topic messages can potentially be accessed by subscribing to wildcards.</p> 
<p><code>{</code></p> 
<p><code>Effect:&nbsp; Allow</code></p> 
<p><code>Action: Subscribe</code></p> 
<p><code>Resource: /topicfilter/building/*</code></p> 
<p><code>Effect: Deny</code></p> 
<p><code>Action: Subscribe</code></p> 
<p><code>Resource: /topicfilter/building/control_room/#</code></p> 
<p><code>Effect: Allow</code></p> 
<p><code>Action: Receive</code></p> 
<p><code>Resource: /topics/building/ *</code></p> 
<p><code>}</code></p> 
<p>However, when a device subscribes to ‘<em>building/#</em>’, it gets messages from ‘<em>building/control_room/3</em>’.</p> 
<p>This is because topic ‘<em>building/#</em>’matches allow ‘<em>building/*</em>’, authorizing the subscription operation for the device. Note that lower in the application code, ‘<em>building/#</em>’ matches all data, and since a device is already subscribed it will receive all the matching topic data.</p> 
<p>When you specify topic filters in AWS IoT Core policies for MQTT clients, MQTT wildcard characters ‘+’ and ‘#’ are treated as literal strings. Their use might result in unintended behavior.</p> 
<p>How to fix it:</p> 
<p><code>{</code></p> 
<p><code>Effect:&nbsp; Allow</code></p> 
<p><code>Action: Subscribe</code></p> 
<p><code>Resource: /topicfilter/building/*</code></p> 
<p><code>Effect: Deny</code></p> 
<p><code>Action: Subscribe</code></p> 
<p><code>Resource: /topicfilter/building/control_room/*</code></p> 
<p><code>Effect: Deny</code></p> 
<p><code>Action: Receive</code></p> 
<p><code>Not-resource: /topic/building/control_room/*</code></p> 
<p><code>}</code></p> 
<p>Once you do that, the device will receive messages published on any topic under topic/ (for example ‘<em>building/common_area</em>’) however, the device will not receive any messages published on any topic under ‘<em>building/control_room/</em>’ (for example ‘<em>building/control_room/3</em>’)</p> 
<p>There could be legitimate use cases where the author may have done it this way, for example, to permit maintenance crew to access a particular space (for example ‘<em>/building/control_room/3</em>’). Thus, in our AWS IoT Device Defender audit check, we named this a potential misconfiguration and we leave it up to the user to decide, whether this was intentional or an unintended misconfiguration.</p> 
<p><strong>Type 2:</strong> When customers want a device to receive messages for a whole topic space ‘<em>building/camera/*</em>’ but not for specific sub-topics that involve control_room as in ‘building/+/control_room’. MQTT wildcards in Deny statements could potentially be circumvented by devices when replacing wildcards with specific strings.</p> 
<p><code>{</code></p> 
<p><code>Effect: Deny</code></p> 
<p><code>Action: Subscribe</code></p> 
<p><code>Resource: /topicfilter/building/+/control_room</code></p> 
<p><code>Effect: Allow</code></p> 
<p><code>Action: Subscribe</code></p> 
<p><code>Resource: /topicfilter/building/camera/*</code></p> 
<p><code>}</code></p> 
<p>The desired behavior is to deny device access to ‘<em>building/camera/control_room</em>’, but allow access to ‘<em>building/camera/resident1</em>’.</p> 
<p>However, devices can send request to topic ‘<em>/building/+/control_room</em>’ and end up receiving messages from topic ‘<em>/building/camera/control_room</em>’.</p> 
<p><strong>How to fix it:</strong></p> 
<p><code>{</code></p> 
<p><code>Effect: Deny</code></p> 
<p><code>Action: Subscribe</code></p> 
<p><code>Resource: /topicfilter/building/*/control_room</code></p> 
<p><code>Effect: Allow</code></p> 
<p><code>Action: Subscribe</code></p> 
<p><code>Resource: &nbsp;/topicfilter/building/camera/*</code></p> 
<p><code>Effect: Allow</code></p> 
<p><code>Action: Receive</code></p> 
<p><code>Resource: /topic/building/camera/*</code></p> 
<p><code>Effect: Deny</code></p> 
<p><code>Action: Receive</code></p> 
<p><code>Resource: /topic/building/*/control_room</code></p> 
<p><code>}</code></p> 
<p>With this fix, IoT policy will allow the device to receive messages from:</p> 
<p><code>/building/camera/resident1</code></p> 
<p><code>/building/camera/resident2</code></p> 
<p><code>/building/camera/resident3</code></p> 
<p>But not from</p> 
<p><code>/building/camera1/control_room</code></p> 
<p><code>/building/camera2/control_room</code></p> 
<p><code>/building/any_camera/control_room</code></p> 
<h2>Identify potential misconfigurations using AWS IoT Device Defender audit check</h2> 
<p>In this section, we’ll show how to configure, run, and take corrective actions in the AWS IoT Console for the two types of misconfigurations described earlier.</p> 
<p>In this example we’ve entered Type 1 and Type 2 in AWS IoT as follows:</p> 
<p><img alt="" class="alignnone size-full wp-image-11863" height="581" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/02/09/figure1.png" width="1527" /></p> 
<p><em>Figure 1: Type 1 policy named as ‘MisconfiguredPolicy’ configured in AWS IoT</em></p> 
<p><img alt="" class="alignnone size-full wp-image-11864" height="582" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/02/09/figure2.png" width="1529" /></p> 
<p><em>Figure 2: Type 2 policy named as ‘MisconfiguredPolicyInfo_2’ configured in AWS IoT</em></p> 
<p>Then, once we run the new ‘IoT policy potentially misconfigured’ audit check, the following reason code is returned when this check finds a potentially misconfigured AWS IoT policy:</p> 
<p>a) POLICY_CONTAINS_MQTT_WILDCARDS_IN_DENY_STATEMENT</p> 
<p>b) TOPIC_FILTERS_INTENDED_TO_DENY_ALLOWED_USING_WILDCARDS</p> 
<p><img alt="" class="alignnone size-full wp-image-11865" height="541" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2023/02/09/figure3.png" width="1591" /></p> 
<p><em>Figure 3: Results from AWS IoT Device Defender ‘IoT policy potentially misconfigured’ audit check</em></p> 
<p>The AWS IoT Device Defender ‘IoT policy potentially misconfigured’ check inspects for MQTT wildcard characters (‘+’&nbsp;or&nbsp;‘#’) in deny statements. Wildcards are treated as literal strings in a policy document and can make it overly permissive.</p> 
<p><strong>How to fix it</strong></p> 
<p>This audit check flags potentially misconfigured policies as there might be false positives. Mark any false positives using <a href="https://docs.aws.amazon.com/iot/latest/developerguide/audit-finding-suppressions.html">Audit finding suppressions</a> so they don’t get flagged in the future.</p> 
<p>You can also follow these steps to fix any noncompliant policies attached to things, thing groups, or other entities:</p> 
<ul> 
 <li>Use <a href="https://alpha.www.docs.aws.a2z.com/iot/latest/apireference/API_CreatePolicyVersion.html">CreatePolicyVersion</a> to create a new, compliant version of the policy. Set the <em>setAsDefault</em> flag to true. (This makes this new version operative for all entities that use the policy.)</li> 
 <li>Verify that all associated devices are able to connect to AWS IoT Core. If a device is unable to connect, use <a href="https://docs.aws.amazon.com/iot/latest/developerguide/audit-chk-iot-misconfigured-policies.html">SetPolicyVersion</a> to roll back the default policy to the previous version, revise the policy, and try again.</li> 
</ul> 
<h2>Conclusion</h2> 
<p>In this post, you’ve learned about finding potential misconfigurations in your IoT policies using AWS IoT Device Defender audit. The ‘IoT policy potentially misconfigured’ audit feature, checks for the use of wild cards in IoT policies and alerts you in the case of potential misconfigurations. Then, you can follow the security recommendations and take corrective actions if needed. This new audit check makes it easier for customers to reduce IoT policy misconfigurations and helps you improve the security posture of your IoT solutions.</p> 
<p>If you use AWS IoT Device Defender, you can enable the new audit check in the AWS IoT Device Defender audit section. If you are new to AWS IoT Device Defender, you can improve the security posture of your IoT device fleet with the one-click process in the AWS IoT console. For more information, refer to AWS IoT Device Defender <a href="https://docs.aws.amazon.com/iot/latest/developerguide/audit-chk-iot-misconfigured-policies.html">documentation</a>.</p> 
<h2>Authors</h2> 
<p><a href="https://www.linkedin.com/in/ryandsouzaaws/">Ryan Dsouza</a> is a Principal Solutions Architect for IoT at AWS. Based in New York City, Ryan helps customers design, develop, and operate more secure, scalable, and innovative solutions using the breadth and depth of AWS capabilities to deliver measurable business outcomes. Ryan has over 25 years of experience in digital platforms, smart manufacturing, energy management, building and industrial automation, and OT/IIoT security across a diverse range of industries. Before AWS, Ryan worked for Accenture, SIEMENS, General Electric, IBM, and AECOM, serving customers for their digital transformation initiatives.</p> 
<p><a href="https://www.linkedin.com/in/andresacaguti/">Andre Sacaguti</a>&nbsp;is a Sr. Product Manager-Tech at AWS IoT. Andre focuses on building products and services that help device makers, automotive manufacturers, and IoT customers from diverse industries to monitor and secure their devices from edge to cloud. Before AWS, Andre built and launched IoT products at T-Mobile and Qualcomm.</p>
