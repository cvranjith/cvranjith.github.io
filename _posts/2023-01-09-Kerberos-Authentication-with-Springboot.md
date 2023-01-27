    Created by Ranjith Vijayan on Jan 08, 2023

## Introduction

Purpose of this document is to give a brief introduction to Kerberos Authentication protocol with a working example that can be useful for application development to integrate with Kerberos solutions. In a typical use-case of Kerberos, there will be a client component, a server component and a Authentication component.

In the example discussed in this document -

* KDC: The Kerberos component is provided as a docker image, so that it can be run in a container that it is platform agnostic. We will set up a basic Kerberos KDC for development purpose (not for production usage).
* Client: The client component is a browser application running in the client machine using a browser.
* Server: A springboot application is written to show the role of a "server" (web-based application) component. This application will have a REST API which requires authentication to serve the resource. We will deploy the springboot application in a docker container.

The document describes all the basic setup of these components. But it will not get into the low level details.

Before getting to the sample set up lets have a quick introduction of Kerberos
What is Kerberos

Kerberos is a Network Authentication Protocol developed at Massachusetts Institute of Technology (MIT). The Kerberos protocol provides a mechanism for mutual authentication between a client and a server before application data is transmitted between them

Windows Server widely supports Kerberos as the default authentication option. Kerberos is a ticket-based authentication protocol that allows nodes in a computer network to identify themselves to each other.

## A Typical Use Case:

Lets say a Client machine wants to access a resource (e.g. a file) from a Server. She goes to the Server and asks, "Hey Server! Can I access the file?"

To protect its resources, Server insists that Client needs to prove her identity. So, when Client asks Server to grant access to the resource, Server says "Umm... but I don't know you, can you first prove your identity?"

Luckily both Client and Server know and trust a common partner called KDC (Key Distribution Center), because both of them have valid memberships with the same KDC. Server tells Client, "Can you go to KDC and ask them to issue a session ticket in your name, addressed to me?"

Client was relieved to hear that, because some time ago she had already proven the identity to KDC by going through the hassles of verification process.

(** A little flashback **) ...

Some time ago, Client had gone to KDC to prove her identity. KDC asked a set of questions (e.g. user-name, password, OTP, Biometric, etc) to ensure that the client is really who she is claiming to be. After the client had promptly answered all the challenges, the KDC gave a certificate called TGT (Ticket Granting Ticket), and told "Here is your TGT and this is valid until tomorrow. If today any server is asking you to prove your identity to them, you can come back to me with this TGT, and tell me the SPN (Service Principal Name) of the server who is asking for your identity. After verification of TGT and membership of both you and the server, I shall give you a one-time-ticket (Session Ticket) which you can present to the server. Don't worry, you will not be grueling with questions again until the expiry of this TGT. Please keep it safe in your wallet!"

(** The flashback ends **)

Client goes to KDC with the valid TGT and told them "Here is my TGT. The Server is asking me to get a session ticket. Can you give me one?"

KDC quickly verifies the validity of TGT, and checks if the memberships of both server and client are active. It then gives the Client, a one-time session ticket which can be presented to the Server.

The client then goes to the server with the session ticket and asks for the resource. This time the server can identify the Client because it had presented the ticket which was issued by the KDC that the server trusts. It however checks the ticket is truly issued by the trusted KDC. For this verification it uses a private key called keytab that was originally issued by the KDC to the Server. The Server checks the session ticket using the keytab and if the verification is success then it can read the identity information of the Client. After successful verification, the server is sure that the client is indeed who she is claiming to be. This process is called Authentication. The server will now apply further checks to determine if the client has the rights to view the file. This check is called Authorization (which is beyond the scope of this document)

In this flow, the Client doesn't have to go through the hassles of sign-on challenges (i.e. provide username password etc) every time it needs to access any server who has a mutual trust with the KDC. This process gives a Single Signon (SSO) experience, which is good for security from the system perspective, and convenient from its users' perspective.






## Setup

Source code for this demo is available in the GIT Repo https://github.com/cvranjith/kerberos-app.git

In this example we are going to run both KDC and server application in Docker Containers. Both these containers expose ports to docker host so that it is accessible via host network.



Following are the values used in demo that you may change in your setup
* KDC	KDC Docker Image	kdc-server:latest	Docker Image
* KDC	KDC ports	88, 749	Exposed to the host machine where the KDC container is running. These values are referred in krb5.conf inside server application configuraiton
* KDC	KDC container name	kdc-server	
* KDC	KDC Host	100.76.135.165	This is the docker host IP address. This value is used in krb5.conf inside server application configuration
* Server Application	Docker Image Name	kerb-auth:1.0	Docker Image
* Server Application	Application Container Name	krb-app	
* Server Application	krb5.conf	<file>	The kerberos config file. KDC server value has to be updated
* Server Applicaiton	Exposed Ports	8080	This is used by the client browser to reach out to the application
* Server Applicaiton	Hostname	fsgbu-mum-266.snbomprshared1.gbucdsint02bom.oraclevcn.com	

This is used by the client browser

This value is also used in KDC configuration to add application server's SPN, and generate keytab file

    
### KDC Component

Lets create the KDC inside a docker container. For this, we will first create the docker image, then we will run the container.


Below are the steps to followed.


    Create Docker image. Save the Dockerfile and init-script.sh into one directory
    Dockerfile

 Expand source

    init-script.sh

 Expand source


  Docker build

 ```
docker build --build-arg http_proxy=http://www-proxy.us.oracle.com:80 --build-arg https_proxy=http://www-proxy.us.oracle.com:80 -t kdc-server:latest .
```

    Run KDC container. (create docker network is optional)
        The port numbers 749 (TCP) and 88 (UDP) are exposed to the host machine.
        By default a realm by name "EXAMPLE.COM" will be created. if you want to change the realm name you can pass the env var -e REALM=YOUR_REALM_NAME.COM

 ```
docker network create example.com
docker run --net example.com -v /dev/urandom:/dev/random -p 749:749 -p 88:88/udp -dit --name kdc-server kdc-server:latest
docker logs kdc-server -f
```

    In the KDC lets Create an entry for the Application Server Principal Name, and also create an user for testing the application.
        In this example the server Principal Name is fsgbu-mum-266.snbomprshared1.gbucdsint02bom.oraclevcn.com and user name is sshuser2

#first go inside the kdc container

```
docker exec -it kdc-server bash
```
    
#Run kadmin command

```
kadmin.local
```
    
# display the existing principlas by typing below command (while you are in kadmin)

```
  listprincs
``` 
 
# Add Principal for the applicaiton Serer

```
  addprinc -randkey host/fsgbu-mum-266.snbomprshared1.gbucdsint02bom.oraclevcn.com@EXAMPLE.COM
``` 
# Save the keytab file

```    
  ktadd -k /fsgbu-mum-266.keytab host/fsgbu-mum-266.snbomprshared1.gbucdsint02bom.oraclevcn.com@EXAMPLE.COM
``` 
# Add principal for the user(s). this will ask you to set password

```    
  addprinc sshuser2
``` 
# Quit kadmin
```
  quit
``` 
# exit docker container
```
    exit
```

The KDC server is now setup.

    We can copy the key-tab file outside of the container (this will be required in the next step

    This is run in the host-machine where the docker container is running. we are saving the keytab file to a directory on the host-machine. the same host-machine will be used to run another container representing the application server.

```
    mkdir -p /tmp/keytab
    chmod a+rwx  /tmp/keytab
    docker cp kdc-server:/fsgbu-mum-266.keytab /tmp/keytab/fsgbu-mum-266.keytab
```

### Application Component (i.e. the server)


    The sample springboot app is available in the git https://github.com/cvranjith/kerberos-app.git. please clone and build (sh mvn clean install )
```
    git clone  https://github.com/cvranjith/kerberos-app.git
    cd <dir>
    sh mvn clean install
```
    Build Docker Image. Copy the Docker file and the built artifact (target/kerb-auth-0.0.1-SNAPSHOT.war) into folder

        Dockerfile
 ```
        FROM openjdk:8-jre-alpine
        COPY kerb-auth-0.0.1-SNAPSHOT.war /kerb-auth-0.0.1-SNAPSHOT.war
        ENTRYPOINT ["sh","-c", "java -jar $JAVA_OPTIONS /kerb-auth-0.0.1-SNAPSHOT.war"]
```
        Build command
```
        docker build -t kerb-auth:1.0 .
```
    Create krb5.conf file. This file contains the information of kerberos KDC server, realms etc. This is to be present in the application server (application container). Since we are going to run the springboot application inside the container, we can create this file and mount it as a volume.
    Create the file /tmp/keytab/krb5.conf (/tmp/keytab is the same directory where the keytab is present)

        The Realm name (EXAMPLE.COM) should match with what we had used for KDC

        The IP address (100.76.135.165 in the below example) is the server where the KDC is running. (since KDC ports are exposed to the docker host, we are using docker-host's IP address here)
```
        [libdefaults]
            default_realm = EXAMPLE.COM
            forwardable = TRUE
         
        [realms]
            EXAMPLE.COM = {
                    kdc_ports = 88
                    kadmind_port = 749
                    kdc = 100.76.135.165
                    admin_server = 100.76.135.165
            }
        [domain_realm]
                100.76.135.165 = EXAMPLE.COM
```
    Run the container

        Please note the file "/tmp/keytab/fsgbu-mum-266.keytab" and "/tmp/keytab/krb5.conf" are mounted

        The keytab file (which is mounted as /keytab/appserver.keytab within the container) needs to be specified to the APP_KEYTAB_LOCATION env var (otherwise you can give this in application.yaml as well)

        The kerberos config file (which is mounted as /tmp/krb.conf) needs to be supplied as java.security.krb5.conf in the JAVA_OPTIONS. Alternatively you can directly mount this file to /etc/krb5.conf which is the default location of java.security.krb5.conf
        the env var APP_SERVICE_PRINCIPAL is to be supplied with the host-name where the applicaiton is running. the same host-name principal should be setup in the KDC. the format for this env var is HTTP/fsgbu-mum-266.snbomprshared1.gbucdsint02bom.oraclevcn.com@EXAMPLE.COM

    
        --net host is given so that the application is accessible on the same port (8080) on the host machine
```
        docker run -dit --name krb-app \
        --net host \
        -v /tmp/keytab/fsgbu-mum-266.keytab:/keytab/appserver.keytab \
        -v /tmp/keytab/krb5.conf:/tmp/krb5.conf \
        -e APP_SERVICE_PRINCIPAL=HTTP/fsgbu-mum-266.snbomprshared1.gbucdsint02bom.oraclevcn.com@EXAMPLE.COM \
        -e APP_KEYTAB_LOCATION=file://keytab/appserver.keytab \
        -e JAVA_OPTIONS=" -Djava.security.krb5.conf=/tmp/krb5.conf " \
        kerb-auth:1.0
```
    Ensure app is running by checking log

        check log
```
        docker logs krb-app -f
```
        You will notice the below lines, that indicate the keytab file is used by the application.
```
        2023-01-07 12:10:46.263 DEBUG 1 --- [           main] w.a.SpnegoAuthenticationProcessingFilter : Filter 'spnegoAuthenticationProcessingFilter' configured for use
        Debug is  true storeKey true useTicketCache false useKeyTab true doNotPrompt true ticketCache is null isInitiator false KeyTab is //keytab/appserver.keytab refreshKrb5Config is false principal is HTTP/fsgbu-mum-266.snbomprshared1.gbucdsint02bom.oraclevcn.com@EXAMPLE.COM tryFirstPass is false useFirstPass is false storePass is false clearPass is false
        principal is HTTP/fsgbu-mum-266.snbomprshared1.gbucdsint02bom.oraclevcn.com@EXAMPLE.COM
        Will use keytab
        Commit Succeeded
      
        2023-01-07 12:10:46.551 DEBUG 1 --- [           main] edFilterInvocationSecurityMetadataSource : Adding web access control expression 'permitAll', for ExactUrl [processUrl='/auth/login?error']
```
    
### Client Component (the browser)


There is nothing as such to set up on the browser.


Please note, in this example we are not using the client browsers capability to use cached ticket of to negotiate the ticket with KDC


In this example we are going to use explicit authentication. so a login page will be displayed for the first time launch of the application URL. Subsequent hits to the application should not give you this


    You can hit the applicaiton URL http://host-name:8080/rest/hello using a browser

    Provide the username and password that we had setup in KDC (in this example sshuser2).
    If everything goes fine it will display the response confirming the user-name as the REST API response as below.


Conclusion


In this chapter we learnt about the time-tested Kerberos authentication protocol, and how to quickly create a dev setup with a Dockerised KDC, and to develop a simple Web Application in Springboot, (also dockerised).

In real world the KDC setup will be very differnt, as it will be usually backed by an LDAP (such as Microsoft Active Directory or Apache Directory etc) for storage of user credentials.

Modern browser can make use of "Integrated Authentication" i.e. when presented with the “Negotiate” challenge, the browser tries to make use of the cached credentials in the host machine, which has already been logged into a KDC principal. In the above example we didnt use this feature, rather we used explicit authentication against the KDC using the login page.


To Do:

    Integrated authentication using Browser's features
    How to use Multiple KDC for authenticaiton
