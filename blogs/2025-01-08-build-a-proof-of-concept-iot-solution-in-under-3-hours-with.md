---
title: "Build a proof-of-concept IoT solution in under 3 hours with the AWS IoT Device Client"
url: "https://aws.amazon.com/blogs/iot/build-a-proof-of-concept-iot-solution-in-under-3-hours-with-the-aws-iot-device-client/"
date: "Wed, 08 Jan 2025 16:57:44 +0000"
author: "Syed Rehan"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-device-defender/feed/"
---
<h2>Introduction</h2> 
<p>You may be starting on your IoT journey, or have thousands of devices connected already. Maybe you just built an IoT business application, and want to deploy it to your fleet. You’re looking for a way to build functionality to control, update, monitor, or secure your IoT devices. To guide you through this process and get you started on AWS IoT, AWS is happy to announce the “Get Started with AWS IoT Workshop”. <a href="https://catalog.workshops.aws/getstartedwithawsiot">Click here to access the Workshop</a>.</p> 
<p>In this hands-on workshop, we use the <a href="https://github.com/awslabs/aws-iot-device-client">AWS IoT Device Client</a> to provide a guided walk-through to create your proof-of-concept IoT project. In <strong>3 hours</strong>, you will learn to:</p> 
<ul> 
 <li>Securely connect your IoT device to the internet, onboard and register it on <a href="https://aws.amazon.com/iot-core/">AWS IoT Core</a></li> 
 <li>Remotely control your device using <a href="https://aws.amazon.com/iot-device-management/">AWS IoT Device Management</a> – run a simple Over-The-Air (OTA) remote operation using Jobs, and set up SSH access for troubleshooting using Secure Tunneling</li> 
 <li>Set up a daily security audit, and monitor a ‘heartbeat’ of health metrics from your device using&nbsp;<a href="https://aws.amazon.com/iot-device-defender/">AWS IoT Device Defender</a></li> 
</ul> 
<p><img alt="" class="size-full wp-image-6143 aligncenter" height="380" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/10/18/image-1.png" width="973" /></p> 
<p>The AWS IoT Device Client is written in C++, open-source, and available on <a href="https://github.com/awslabs/aws-iot-device-client">GitHub</a>. You can compile and install on Embedded-Linux based IoT devices to get started with AWS IoT Core, AWS IoT Device Management, and AWS IoT Device Defender.</p> 
<h2>Prerequisites</h2> 
<p>To complete this workshop, you need:</p> 
<ul> 
 <li>An AWS account with admin privileges, or Event engine details. You can <a href="https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/">create a new AWS account here</a>.</li> 
 <li>A computer with the latest browser – like Firefox or Chrome</li> 
 <li>Basic understanding of Linux (e.g. create directories, set file permissions) and programming (compiling code)</li> 
</ul> 
<h2>When to use the AWS IoT Device Client</h2> 
<h3>Example Use Cases:</h3> 
<p>The AWS IoT Device Client is a reference implementation, and the easiest way to create an IoT proof-of-concept (PoC). It provides an easy way to connect a fleet of devices to the internet, and route IoT data to AWS. By default, it enables you to operate, manage, and control your fleets, or secure them against threats using AWS IoT services. It is open-source, so you can modify it to fit your business needs, connect your business applications to take advantage of AWS IoT features, or optimize its resource utilization when you wish to scale up from a PoC to production. Here are some example use cases the AWS IoT Device Client solves for:</p> 
<ol> 
 <li><span style="text-decoration: underline;">[<strong>First Connect &amp; Provisioning</strong>] You want to provision a fleet of production devices and connect them to the internet.</span><br /> The IoT Device Client enables your devices to automatically connect to IoT Core, exchange a bulk certificate for secure individual identities from the <a href="https://aws.amazon.com/iot-core/features/#Authentication_and_Authorization">IoT Core Identity</a> service, and register themselves in the <a href="https://aws.amazon.com/iot-core/features/#Registry">IoT Core Device Registry</a>.</li> 
 <li>You just built a custom business application for your IoT solution. The IoT Device Client provides a backbone of capabilities for your app. 
  <ol type="a"> 
   <li><span style="text-decoration: underline;">[<strong>Messaging</strong>] You want to exchange telemetry, state, or control messages with the app over MQTT.</span><br /> The IoT Device Client enables your device connect over MQTT to the <a href="https://aws.amazon.com/iot-core/features/#Device_Gateway">AWS IoT Core Device Gateway</a> and shares that connection with your app. You can publish/subscribe to custom MQTT topics via the <a href="https://aws.amazon.com/iot-core/features/#Message_Broker">AWS IoT Core Message Broker</a> by setting simple configurations on your device. You also have the option to publish data from your app directly to the <a href="https://aws.amazon.com/iot-core/features/#Rules_Engine">AWS IoT Core Rules Engine</a> via <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-basic-ingest.html">Basic Ingest</a>, reducing messaging costs.</li> 
   <li><span style="text-decoration: underline;">[<strong>Control</strong>] You want to read and control the state of your device or the configuration of your app.</span><br /> The IoT Device Client gives your app the ability to interact with <a href="https://aws.amazon.com/iot-core/features/#Device_Shadow">AWS IoT Core Device Shadows</a> so you can get/set the state of your device or the configuration of your app even if it is offline for prolonged periods.</li> 
   <li><span style="text-decoration: underline;">[<strong>Operate &amp; Update</strong>] You want to update your fleet to use a new version of your app, or deploy a firmware/OS update, or simply reboot the fleet remotely.</span><br /> With the IoT Device Client, you can directly use <a href="https://aws.amazon.com/iot-device-management/features/#:~:text=Remotely%20Manage%20Connected%20Devices">AWS IoT Device Management Jobs</a> – it lets you deploy to targeted devices, control the speed of your deployment, and track the status of your updates, even if devices work in partially offline environments.</li> 
   <li><span style="text-decoration: underline;">[<strong>Troubleshoot or Access</strong>] You want to troubleshoot a device, retrieve logs, or access it using Secure Shell (SSH) for maintenance.</span><br /> With the IoT Device Client your device can directly connect using the <a href="https://aws.amazon.com/iot-device-management/features/#:~:text=to%20connected%20devices.-,Secure%20Tunneling,-AWS%20IoT%20Device">AWS IoT Device Management Secure Tunneling</a> feature to an Admin console, providing synchronous access with admin privileges.</li> 
   <li><span style="text-decoration: underline;">[<strong>Monitor &amp; Secure</strong>] You want to send a ‘heartbeat’ of device-side health metrics like ports open or bytes in/out to detect unusual security behaviors and guard your fleet against compromise. </span><br /> The IoT Device Client lets your device automatically publish your metrics over MQTT to the <a href="https://aws.amazon.com/iot-device-defender">AWS IoT Device Defender</a> service at regular intervals.</li> 
  </ol> </li> 
</ol> 
<h2>AWS IoT Device Client: High Level Architecture</h2> 
<p><img alt="" class="size-full wp-image-6144 aligncenter" height="256" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/10/18/image-2.png" width="615" /></p> 
<h3>Compatibility:</h3> 
<p>The AWS IoT Device Client [<a href="https://github.com/awslabs/aws-iot-device-client">GitHub</a>] currently works on IoT devices with common microprocessors (x86_64, ARM, MIPS-32 architectures), and common Linux software environments (Debian, Ubuntu, and RHEL). We also provide a <a href="https://github.com/aws4embeddedlinux/meta-aws/tree/master/recipes-iot/aws-iot-device-client">meta-aws recipe for the AWS IoT Device Client</a> that you can build into your Yocto Linux distribution for more constrained and purpose-built devices.</p> 
<h2>Conclusion</h2> 
<p>Try out this <a href="https://catalog.us-east-1.prod.workshops.aws/v2/workshops/6d30487a-48e1-4631-b6bc-5602582800b5/en-US">Workshop</a> to get started with AWS IoT using the AWS IoT Device Client.</p> 
<p>Using <strong>AWS IoT Device Client</strong> is the easiest way to create a proof-of-concept (PoC) for your IoT project. It takes away the generic heavy lifting involved in connecting, managing, and securing your IoT fleets, reducing the initial investment required for your IoT project. You can now focus on building your IoT business logic and apps. AWS is committed to the AWS IoT Device Client as a living tool. It is a reference implementation with operational and security best-practices baked in. As new AWS IoT features become generally available and IoT best practices are established, we will update this software to support them appropriately.</p> 
<h2><strong>About the authors</strong></h2> 
<p>
 <!-- First Author --></p> 
<div class="blog-author-box" style="border: 1px solid #d5dbdb; padding: 15px;"> 
 <p class="NAME OF YOUR IMAGE FROM MEDIA LIBRARY"><img alt="syed" class="wp-image-16165 size-full alignleft" height="121" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2024/10/01/Syed125px.jpg" width="125" /></p> 
 <h3 class="lb-h4"><a href="https://www.linkedin.com/in/iamsyed/">Syed Rehan</a></h3> 
 <p>Syed is a Senior IoT Product Security Architect at AWS IoT. He specializes in enabling customers—from startups to large enterprises—to build secure IoT, Machine Learning (ML), and Artificial Intelligence (AI)-based solutions on AWS. With deep expertise in cybersecurity, cloud technologies, and IoT, Syed collaborates with security specialists, developers, and decision-makers to drive the adoption of AWS Security services and solutions. Before AWS, Syed designed and developed mission-critical systems for companies like Vodafone, FICO, Rackspace, Nokia, Barclays Bank, and Convergys. He is also a published author on AWS IoT, ML, and Cybersecurity, sharing his knowledge through books and public speaking engagements.</p> 
</div> 
<p>
 <!-- Second Author --></p> 
<div class="blog-author-box" style="border: 1px solid #d5dbdb; padding: 15px;"> 
 <p class="NAME OF YOUR IMAGE FROM MEDIA LIBRARY"><img alt="" class="wp-image-16429 size-full alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/10/18/shantanu.jpeg" width="120" /></p> 
 <h3 class="lb-h4"><a href="https://www.linkedin.com/in/satheshantanu/">Shantanu Sathe</a></h3> 
 <p style="color: #000000;">is a Senior Product Manager – Technical at AWS IoT. He focuses on building IoT fleet management and monitoring solutions.</p> 
</div>
