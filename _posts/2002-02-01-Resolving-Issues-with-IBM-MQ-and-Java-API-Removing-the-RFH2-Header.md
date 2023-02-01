    Created by Ranjith Vijayan on Feb 01, 2023

When publishing messages to IBM MQ using a Java client, traditional IBM MQ applications may have trouble reading the messages. This is often due to the "RFH2" header that is included in the message, which carries JMS-specific information.

The issue is usually related to the "TargetClient" configuration. IBM MQ messages are made up of three components:

    The IBM MQ Message Descriptor (MQMD)
    An IBM MQ MQRFH2 header
    The message body

The MQRFH2 header is optional and its inclusion is controlled by the "TARGCLIENT" flag in the JMS Destination class. This flag can be set using the IBM MQ JMS administration tool. When the recipient is a JMS application, the MQRFH2 header should always be included. However, when sending directly to a non-JMS application, the header should be omitted as these applications do not expect it in the IBM MQ message.

### What is RHF2 Header
The "RFH2" header (Reply-To-Format header) is an IBM MQ header used for routing JMS messages. It contains information about the format of the message such as its encoding, character set, and data format. The header allows messages to be properly processed by different systems and specifies the format of reply messages.


### What is Target Cleint

"IBM MQ Target Client" is a component of IBM MQ, a messaging and integration middleware solution. It is a software library that allows applications to connect to IBM MQ and receive messages. The Target Client provides a high-performance, reliable, and secure way for applications to receive messages and supports multiple programming languages.

The relationship between RFH2 headers and the IBM MQ Target Client is that the Target Client is used to receive messages from IBM MQ queues, and the RFH2 headers are used to route and format the messages within IBM MQ. The Target Client uses the information in the RFH2 headers to properly format and present the messages to the application.


### How to Remove RFH2 Header while using Java clients

To remove the RFH2 header, one of the below two methods can be employed


1. Using TC(MQ)
  While creating the queue (on IBM MQ side), add the keyword "TC(MQ)". This will ensure that any client, including the Java Client, will use the Target Client as MQ, which is suitable for traditional clients.
  
  ```
  define Q(MyJMSQueueName) queue(MYMQ.Q) TC(MQ)
  ```
  
2. using URL Parameter "targetClient=1"
    Use the URL parameter "targetClient=1" in the queue name. This option is useful if you don't have control over how the MQ administrator creates the queue, but you can set the preference for targetClient to "1" (for MQ) when connecting to the queue as a client.
    
  ```
  queue:///MYQUEUE?targetClient=1
  ```


Reference: https://www.ibm.com/docs/en/ibm-mq/8.0?topic=messages-mapping-jms-onto-mq
