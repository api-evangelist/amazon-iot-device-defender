---
title: "Monitor AWS IoT connections in near-real time using MQTT LWT"
url: "https://aws.amazon.com/blogs/iot/monitor-aws-iot-connections-in-near-real-time-using-mqtt-lwt/"
date: "Fri, 13 May 2022 19:55:02 +0000"
author: "Syed Rehan"
feed_url: "https://aws.amazon.com/blogs/iot/tag/aws-iot-device-defender/feed/"
---
<p>In a connected device, you may need to monitor devices in near-real time to detect error and mitigate actions, Last Will and Testament (LWT) method for MQTT addresses this challenge. LWT is a standard method of MQTT protocol specification that allows to detect abrupt disconnects of devices and to notify other clients about this abrupt disconnections.</p> 
<p>IoT devices are often used in environments with unreliable network connectivity and/or devices might disconnect due to lack of power supply, low battery, loss of connection, or any other reason. This will cause abrupt disconnections from the broker without knowing if the disruption was forced by the client or truly abrupt, This is where LWT let’s a client provide a testament along with its credentials when connecting to the AWS IoT Core. If the client disconnects abruptly at some point later (i.e. power loss), it can let AWS IoT Core deliver a message to other clients and inform them of this abrupt disconnect and deliver LWT message.</p> 
<p>MQTT Version 3.1.1 provides an LWT feature as part of the MQTT message and is supported by <a href="https://aws.amazon.com/iot-core/">AWS IoT Core</a>, so any client which disconnects abruptly can specify its LWT message along with the MQTT topic when it connects to the broker. When the client disconnects abruptly, the broker (AWS IoT Core) will then publish the LWT message provided by that client at connection time to all the devices which subscribed to this LWT topic.</p> 
<p>The MQTT LWT feature enables you to monitor AWS IoT connections in near-real time to help you to take corrective actions. You can react to abrupt disconnection events by verifying status, restoring connections, and carrying out either edge-based (device side) actions or cloud-based actions to investigate and mitigate this abrupt disconnect of the device.</p> 
<p>In this blog we will go through following steps:</p> 
<ol> 
 <li>A simulated ‘lwtThing’ device connects to AWS IoT Core by giving Keep-alive time</li> 
 <li>The ‘lwtThing’ device, on the connection to AWS IoT Core, provides the following: 
  <ol> 
   <li>Topic for LWT (i.e. /last/will/topic)</li> 
   <li>LWT message</li> 
   <li>QoS type either 0 or 1</li> 
  </ol> </li> 
 <li>‘lwtThing’ device disconnects abruptly from AWS IoT Core</li> 
 <li>AWS IoT Core detects this and publishes the LWT message to all the subscribers of the topic (i.e. /last/will/topic)</li> 
 <li>Rules for AWS IoT (rule engine) picks up the trigger on the topic and invokes <a href="https://aws.amazon.com/sns/?whats-new-cards.sort-by=item.additionalFields.postDateTime&amp;whats-new-cards.sort-order=desc">Amazon Simple Notifications Service (SNS)</a></li> 
 <li>Amazon SNS sends a notification email</li> 
</ol> 
<p>We will setup a virtual environment using a CloudFormation template (by using <a href="https://catalog.us-east-1.prod.workshops.aws/workshops/6d30487a-48e1-4631-b6bc-5602582800b5/en-US/">AWS IoT workshop setup instructions</a>) and launch a virtual IoT thing (naming ‘lwtThing’) to create a real life simulation of the physical device.</p> 
<h2 style="text-align: left;">Architecture</h2> 
<p><img alt="" class="alignnone size-full wp-image-7724" height="345" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/11/ArchLWT.png" width="805" /></p> 
<p>We will simulate the edge device using a script provided below and send the LWT message, showing abrupt disconnects and triggering AWS IoT rules and subsequently invoking Amazon SNS to send emails.</p> 
<h2>Setup</h2> 
<p>We will use the following workshop setup to get quickly bootstrapped and test LWT. You can use the following <a href="https://catalog.us-east-1.prod.workshops.aws/workshops/6d30487a-48e1-4631-b6bc-5602582800b5/en-US/chapter2-lets-begin/10-step1">link</a> to setup AWS Cloud9 environment (pick any region closest to your location).</p> 
<p>Once we have the environment setup using the workshop AWS CloudFormation pre-provided template, lets begin testing the ungraceful disconnects with AWS IoT Core (AWS MQTT broker on the cloud).</p> 
<p>Now open the Cloud9 terminal (see <a href="https://catalog.us-east-1.prod.workshops.aws/workshops/6d30487a-48e1-4631-b6bc-5602582800b5/en-US/chapter2-lets-begin/10-step1#opening-a-terminal-in-cloud9">here</a>) and let’s setup Python SDK for us to use.</p> 
<p>Create a folder for us to use to connect our IoT thing using the Cloud9 terminal window.</p> 
<pre><code class="lang-bash">mkdir -p /home/ubuntu/environment/lwt/certs
cd /home/ubuntu/environment/lwt/</code></pre> 
<p>Setup Python IoT SDK using full instructions <a href="https://github.com/aws/aws-iot-device-sdk-python#installation">here</a>.</p> 
<p>Quick instructions:</p> 
<pre><code class="lang-git">git clone https://github.com/aws/aws-iot-device-sdk-python.git
cd aws-iot-device-sdk-python
python setup.py install
</code></pre> 
<p>Now, to setup your AWS IoT Thing follow steps outlined <a href="https://catalog.us-east-1.prod.workshops.aws/workshops/6d30487a-48e1-4631-b6bc-5602582800b5/en-US/chapter3-deviceclientsetup/10-dc-setup">here</a>.</p> 
<p>Once we have created the thing, let’s upload these certificates in our Cloud9 instance for us to connect from there.</p> 
<p>Upload the newly created certificates and RootCA into following folder (created earlier)</p> 
<pre><code class="lang-bash">/home/ubuntu/environment/lwt/certs</code></pre> 
<h2>LWT thing messages</h2> 
<p>Let’s copy the Python code to Cloud9 and execute as the simulated AWS IoT thing.</p> 
<p>Copy the following commands:</p> 
<pre><code class="lang-bash">touch lwtTest.py</code></pre> 
<p>Open the file and copy the following code into it.</p> 
<pre class="unlimited-height-code"><code class="lang-python">'''
/*
 * # Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
 * # SPDX-License-Identifier: MIT-0
 * 
 */


 '''
from AWSIoTPythonSDK.MQTTLib import AWSIoTMQTTClient
import logging
import time
import argparse
import json

AllowedActions = ['both', 'publish', 'subscribe']

# Custom MQTT message callback
def customCallback(client, userdata, message):
    print("Received a new message: ")
    print(message.payload)
    print("from topic: ")
    print(message.topic)
    print("--------------\n\n")

# LWT JSON payload
payload ={
  "state": {
    "reported": {
      "last_will": "yes",
      "trigger_action": "on",
      "client_id": "lwtThing"
        }
    }
}
 
# conversion to JSON done by dumps() function
jsonPayload = json.dumps(payload)
 
# printing the output
#print(jsonPayload)


# Read in command-line parameters
parser = argparse.ArgumentParser()
parser.add_argument("-e", "--endpoint", action="store", required=True, dest="host", help="Your AWS IoT custom endpoint")
parser.add_argument("-r", "--rootCA", action="store", required=True, dest="rootCAPath", help="Root CA file path")
parser.add_argument("-c", "--cert", action="store", dest="certificatePath", help="Certificate file path")
parser.add_argument("-k", "--key", action="store", dest="privateKeyPath", help="Private key file path")
parser.add_argument("-p", "--port", action="store", dest="port", type=int, help="Port number override")
parser.add_argument("-w", "--websocket", action="store_true", dest="useWebsocket", default=False,
                    help="Use MQTT over WebSocket")
parser.add_argument("-id", "--clientId", action="store", dest="clientId", default="basicPubSub",
                    help="Targeted client id")
parser.add_argument("-t", "--topic", action="store", dest="topic", default="sdk/test/Python", help="Targeted topic")
parser.add_argument("-m", "--mode", action="store", dest="mode", default="both",
                    help="Operation modes: %s"%str(AllowedActions))
parser.add_argument("-M", "--message", action="store", dest="message", default="AWS IoT Thing connected message to IoT Core",
                    help="Message to publish")

args = parser.parse_args()
host = args.host
rootCAPath = args.rootCAPath
certificatePath = args.certificatePath
privateKeyPath = args.privateKeyPath
port = args.port
useWebsocket = args.useWebsocket
clientId = args.clientId
topic = args.topic

if args.mode not in AllowedActions:
    parser.error("Unknown --mode option %s. Must be one of %s" % (args.mode, str(AllowedActions)))
    exit(2)

if args.useWebsocket and args.certificatePath and args.privateKeyPath:
    parser.error("X.509 cert authentication and WebSocket are mutual exclusive. Please pick one.")
    exit(2)

if not args.useWebsocket and (not args.certificatePath or not args.privateKeyPath):
    parser.error("Missing credentials for authentication.")
    exit(2)

# Port defaults
if args.useWebsocket and not args.port:  # When no port override for WebSocket, default to 443
    port = 443
if not args.useWebsocket and not args.port:  # When no port override for non-WebSocket, default to 8883
    port = 8883

# Configure logging - we will see messages on STDOUT
logger = logging.getLogger("AWSIoTPythonSDK.core")
logger.setLevel(logging.DEBUG)
streamHandler = logging.StreamHandler()
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
streamHandler.setFormatter(formatter)
logger.addHandler(streamHandler)

# Init AWSIoTMQTTClient
myAWSIoTMQTTClient = None
if useWebsocket:
    myAWSIoTMQTTClient = AWSIoTMQTTClient(clientId, useWebsocket=True)
    myAWSIoTMQTTClient.configureEndpoint(host, port)
    myAWSIoTMQTTClient.configureCredentials(rootCAPath)
else:
    myAWSIoTMQTTClient = AWSIoTMQTTClient(clientId)
    myAWSIoTMQTTClient.configureEndpoint(host, port)
    myAWSIoTMQTTClient.configureCredentials(rootCAPath, privateKeyPath, certificatePath)

#########
# Will Topic
# Input parameters are: Topic, Last will message and finally QoS
myAWSIoTMQTTClient.configureLastWill('/last/will/topic', jsonPayload, 0)
#########


# Connect and subscribe to AWS IoT
# keep-alive connect parameter - setting 30s
myAWSIoTMQTTClient.connect(30) 
print("Connected!")
loopCount = 1
while loopCount &lt; 2:
    if args.mode == 'both' or args.mode == 'publish':
        message = {}
        message['message'] = args.message
        messageJson = json.dumps(message)
        myAWSIoTMQTTClient.publish(topic, messageJson, 1)
        if args.mode == 'publish':
            print('Published topic %s: %s\n' % (topic, messageJson))
            loopCount +=1
#lets put the device to sleep so it creates disconnect after 60s
print("--- Putting device to sleep now, so IoT core keep-alive time expires. ---")
print("--- We will abruptly disconnect the device after 60seconds. ---")
time.sleep(60)

</code></pre> 
<p>Let’s look at the following line which is doing all the work on setting the LWT Topic, JSON payload, and what level of QoS we are using.</p> 
<pre><code class="lang-json">myAWSIoTMQTTClient.configureLastWill('/last/will/topic', jsonPayload, 0)</code></pre> 
<ul> 
 <li>Topic used is : /last/will/topic</li> 
 <li>QoS (Quality of Service) is: 0</li> 
 <li>JSON Payload variable contains following payload:</li> 
</ul> 
<pre class="unlimited-height-code"><code class="lang-json">{
  "state": {
    "reported": {
      "last_will": "yes",
      "trigger_action": "on",
      "client_id": "lwtThing"
        }
    }
}</code></pre> 
<p>The above setup defines the LWT topic as well as what topic to post this message to, which will be understood and executed by AWS IoT rules once the device disconnects abruptly (The “Last Will” is<br /> published by the server when its connection to the client is unexpectedly lost.) An AWS IoT rule will trigger the action on Amazon SNS to send an email upon its execution. You can read more on the other options in the <a href="https://s3.amazonaws.com/aws-iot-device-sdk-python-docs/sphinx/html/index.html">SDK document</a>.</p> 
<p>We are setting keep-alive to 30seconds at connection to AWS IoT core so it keeps the session alive for the given time. Once the time runs out, the session is expired.</p> 
<p>At the expiration of the session, we set the device to sleep for 60 seconds, Once 60 seconds finishes we abruptly disconnects the devices which in turn generates Last Will Testament (LWT) trigger from AWS IoT Core and message gets published to all topic subscribers who are listening to this LWT topic.</p> 
<h2>Setup Amazon SNS</h2> 
<p>Let’s setup Amazon SNS and configure it to send email as its notification, From the <a href="https://console.aws.amazon.com/sns">Amazon SNS console</a> do the following:</p> 
<ul> 
 <li>Select Topics 
  <ul> 
   <li>Select <strong>Create topic</strong> 
    <ul> 
     <li>Select <strong>Standard</strong></li> 
     <li>Select <strong>Name</strong> (i.e. lwtSNSTopic)</li> 
     <li>Select <strong>Display name</strong> (i.e. lwtSNSTopic)</li> 
     <li>Select <strong>Create topic</strong></li> 
    </ul> </li> 
   <li>Once topic is created 
    <ul> 
     <li>Select <strong>Create subscription</strong></li> 
     <li>Select <strong>Email</strong> from Protocol dropdown</li> 
     <li>For <strong>Endpoint</strong> give the email address you would like to use</li> 
     <li>Select <strong>Create subscription</strong></li> 
    </ul> </li> 
  </ul> </li> 
</ul> 
<p>You should receive an email. Please confirm the subscription. If you have not confirmed the subscription, you will not be able to receive any emails.</p> 
<h2>Setup Rules for AWS IoT Core</h2> 
<p>From the <a href="https://console.aws.amazon.com/iot/">AWS IoT Core console</a> do the following:</p> 
<ul> 
 <li>Select Act 
  <ul> 
   <li>Select <strong>Rules</strong></li> 
   <li>Select <strong>Create</strong></li> 
   <li>Give a <strong>name</strong> (i.e. lastWillRule) and <strong>description</strong> (My first LWT rule)</li> 
   <li>In <strong>Rule query statement</strong> enter following: 
    <ul> 
     <li><code>SELECT * FROM '/last/will/topic' where state.reported.last_will = 'yes' and state.reported.trigger_action = 'on'</code></li> 
    </ul> </li> 
   <li>In Actions section 
    <ul> 
     <li>Select <strong>Add Action</strong></li> 
     <li>Select <strong>Send a message to an SNS push notification</strong></li> 
     <li>Select <strong>Configure action</strong></li> 
     <li>In SNS target <strong>Select</strong> the SNS topic you created earlier (i.e. lwtSNSTopic)</li> 
     <li>In Message format, Select <strong>JSON</strong></li> 
     <li>Select <strong>Create Role</strong></li> 
     <li>Give it a name (i.e. lwtRuleRole)</li> 
     <li>Select <strong>Add action</strong></li> 
    </ul> </li> 
  </ul> </li> 
</ul> 
<p>Let’s add another action here, we will republish the incoming LWT message to another topic to verify its incoming.</p> 
<ul> 
 <li> 
  <ul> 
   <li>In Actions section 
    <ul> 
     <li>Select <strong>Add Action</strong></li> 
     <li>Select <strong>Republish a message to an AWS IoT topic</strong></li> 
     <li>Select <strong>Configure action</strong></li> 
     <li>Under Topic 
      <ul> 
       <li>Select <strong>/lwt/executed</strong></li> 
       <li>we can leave the <strong>Quality of Service</strong> default</li> 
       <li>For ‘Choose or create a role to grant AWS IoT access to perform this action 
        <ul> 
         <li>Select <strong>lwtRuleRole</strong></li> 
         <li>Select <strong>Update role</strong></li> 
        </ul> </li> 
       <li>Select <strong>Add action</strong></li> 
      </ul> </li> 
    </ul> </li> 
  </ul> </li> 
</ul> 
<p>This concludes our rules setup section, let’s proceed and setup sending LWT messages and execute our setup.</p> 
<h2>Sending LWT messages</h2> 
<p>Before we execute the simulated device (using python code) let’s subscribe to the topic in the <a href="https://console.aws.amazon.com/iot/">AWS IoT Core console</a>.</p> 
<p><img alt="" class="alignnone size-large wp-image-7727" height="460" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/11/Screenshot-2022-03-21-at-17.08.44-1024x460.png" width="1024" /></p> 
<p style="text-align: center;">Figure 2</p> 
<p>Now that we have everything in place, let’s execute the IoT Thing (simulated using Python code). You can use the sample execution command which may differ for you as your thingID might be different or your certificates path might be in a different location.</p> 
<p><strong>Sample command (replace xxxx with relevant values for your setup):</strong></p> 
<pre class="unlimited-height-code"><code class="lang-bash">python lwtTest.py -e xxxxxxxxxxxxxx-ats.iot.us-east-1.amazonaws.com -r /home/ubuntu/environment/lwt/certs/AmazonRootCA1.pem -c /home/ubuntu/environment/lwt/certs/xxxxxxxxxxxxxxxxxxxxxxxxxxxx-certificate.pem.crt -k /home/ubuntu/environment/lwt/certs/xxxxxxxxxxxxxxxxxxxxxxxxxxxx-private.pem.key -id lwtThing -t /lwt/connected/topic -m publish</code></pre> 
<p>What we are passing as input parameters to the code is as follows:</p> 
<ul> 
 <li>-e is referring to the end point of AWS IoT Core</li> 
 <li>-r is the full file path where our Amazon Root CA is located</li> 
 <li>-c is the full file path for our certificate location</li> 
 <li>-k is the full file path for our private key</li> 
 <li>-id is the ClientID we are using to send to AWS IoT Core (you should match this to what you have created the Thing in IoT Core as)</li> 
 <li>-t is the topic we are providing to publish on when it first connects to AWS IoT Core</li> 
 <li>-m is the mode we have defined in the code and we will use publish for this test. (available modes are: publish, subscribe or both)</li> 
</ul> 
<p>Let’s look at the execution of the command, we should see that LWT is getting configured and what message we published to AWS IoT Core. You will also see abrupt disconnect after 60 seconds.</p> 
<p><img alt="" class="alignnone size-large wp-image-7728" height="336" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/11/LWTBlog2-1024x336.png" width="1024" /></p> 
<p style="text-align: center;">Figure 3</p> 
<p>Switching over to the AWS IoT Core console to see incoming messages, subscribe to following topics:</p> 
<ul> 
 <li>Topic used for republishing of the message when the rule is executed (using as debug): <code>/lwt/executed</code></li> 
 <li>Topic used for when LWT message is published upon ungraceful disconnect of a client: <code>/last/will/topic</code></li> 
 <li>Topic <code>/lwt/connected/topic</code> you can see messages posted by the thing. This occurs when the client is connected to AWS IoT Core and sends the message to inform the broker I’m here and connected.</li> 
</ul> 
<p><img alt="" class="alignnone size-large wp-image-7729" height="472" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/11/Screenshot-2022-03-24-at-22.40.36-1024x472.png" width="1024" /></p> 
<p style="text-align: center;">Figure 4</p> 
<p>Under topic <code>/last/will/topic</code> we can see the message executed by AWS IoT Core once the device ungracefully disconnects.</p> 
<p><img alt="" class="alignnone size-large wp-image-7730" height="459" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/11/Screenshot-2022-03-24-at-22.41.54-1024x459.png" width="1024" /></p> 
<p style="text-align: center;">Figure 5</p> 
<p>When AWS IoT rule is executed for LWT we can see within topic <code>/lwt/executed</code> payload is published to this topic too, we configured this topic earlier to repost to when AWS IoT rule is executed upon device abrupt disconnection.</p> 
<p><img alt="" class="alignnone size-large wp-image-7731" height="528" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/11/Screenshot-2022-03-24-at-22.44.10-1024x528.png" width="1024" /></p> 
<p style="text-align: center;">Figure 6</p> 
<p>Upon successful execution of the AWS IoT rule we also triggered Amazon SNS email notification and if you have configured this correctly earlier you will see similar email in your inbox.</p> 
<p><img alt="" class="alignnone size-large wp-image-7732" height="172" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2022/05/11/Screenshot-2022-03-24-at-22.50.23-1024x172.png" width="1024" /></p> 
<p style="text-align: center;">Figure 7</p> 
<h2>Conclusion</h2> 
<p>In this blog we looked at how you can use AWS IoT Core to detect errors and failures of a device and abrupt disconnections, and upon abrupt disconnection triggering Amazon SNS email notification to support team who can quickly investigate and mitigate failure and resolve issues at large. If the thing closes connection properly or in a recommended manner, then AWS IoT Core will disregard the LWT which we set at the time of connection. By using LWT, we can implement many error handling scenarios where the connection of the client drops and where there is a dependency of other clients relying on this connection chain. For example, when an industrial gateway responsible for gathering sensor data across the factory floor experiences an abrupt disconnection from AWS IoT Core, then you can monitor those disconnections and take corrective measures to reduce second degree impact downstream. You can read more here about <a href="https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html">MQTT</a> and <a href="https://aws.amazon.com/sns/features/">SNS</a>.</p> 
<h2></h2> 
<h2>About the author</h2> 
<p><img alt="" class="size-full wp-image-6257 alignleft" height="143" src="https://d2908q01vomqb2.cloudfront.net/f6e1126cedebf23e1463aee73f9df08783640400/2021/11/19/syedPic_adobespark.jpeg" width="107" /><a href="https://www.linkedin.com/in/iamsyed/">Syed Rehan</a> is a Sr. Global specialist Solutions Architect at Amazon Web Services (AWS) and is based out of London. He is covering global span of customers and supporting them as lead IoT Solution Architect. Syed has in-depth knowledge of IoT and cloud and works in this role with global customers ranging from start-up to enterprises to enable them to build IoT solutions with the AWS eco system.</p>
