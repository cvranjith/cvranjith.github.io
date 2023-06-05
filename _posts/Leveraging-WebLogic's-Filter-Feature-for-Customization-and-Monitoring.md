# Leveraging WebLogic's Filter Feature for Customization and Monitoring


WebLogic Server provides a powerful feature called filters that allows developers and administrators to customize request processing and enhance monitoring capabilities without modifying the underlying code of a web application. In this article, we'll explore the benefits of WebLogic's filter feature and delve into two common use cases: adding custom HTTP headers and adding additional custom logging for monitoring purposes. This can be achieved without modifying the underlying source code of the web-application

## The Power of Filters

Filters in WebLogic Server act as intermediaries between clients and web resources, allowing for pre-processing and post-processing of HTTP requests and responses. They provide a way to inject custom logic into the request processing pipeline without the need to modify the application code itself. This separation of concerns enables developers to add or modify functionality at the server level, offering greater flexibility and ease of maintenance.

## Use Case 1: Adding Custom HTTP Headers

One common use case for WebLogic filters is the addition of custom HTTP headers to incoming requests. Custom headers can carry valuable metadata or control information that can be utilized by downstream systems or monitoring tools. For example, an organization might want to add a unique identifier or correlation ID to each request for tracing and troubleshooting purposes. By implementing a custom filter, this header can be automatically appended to every incoming request, providing valuable contextual information throughout the application stack.

To achieve this, we can create a custom filter in WebLogic Server and configure it to intercept incoming requests. Within the filter's logic, we can manipulate the request object to add the desired custom headers before passing it along the filter chain for further processing. The beauty of this approach is that the core application code remains untouched, making it easy to introduce or remove such functionality as needed.

## Use Case 2: Additional Custom Logging for Monitoring

Another common use case for WebLogic filters is enhancing the logging capabilities of a web application for monitoring purposes. While most application servers provide built-in logging features, they may not cover all the specific requirements of an organization. In such cases, filters can be leveraged to augment the existing logging mechanism with additional custom log entries that capture specific information or metrics.

For instance, let's say we want to log detailed information about incoming requests and outgoing responses, including the elapsed time for each request. By implementing a custom filter, we can capture the necessary data at the entry and exit points of request processing, calculate the elapsed time, and log it along with other relevant request and response information. This enriched logging can provide valuable insights for performance analysis, troubleshooting, and auditing purposes.

Similar to the previous use case, the ability to add custom logging without modifying the application code simplifies maintenance and allows for seamless integration with existing monitoring and log analysis tools.


## Example Configuration

Sample Source CustomHeaderFilter.java is available in the wls-filter repo of my gitHub.

As per this, filter class file is created that can set response headers as defined in web.xml

Below are the steps to follow

### 1. Create Jar file containing custom filter

``` bash
## since I am running wls inside a container, first copying the file inside container.
docker cp CustomHeaderFilter.java wls:/tmp
docker exec -it wls bash
 
## Prepare sourcce
rm -rf /tmp/filter
mkdir -p /tmp/filter/bin/com/custom/filter/
mkdir -p /tmp/filter/src/com/custom/filter/
cp /tmp/CustomHeaderFilter.java /tmp/filter/src/com/custom/filter/
## execute setDomainEnv.sh
source /u01/oracle/user_projects/domains/jmstest/bin/setDomainEnv.sh
cd /tmp/filter/
touch manifest.txt
## compile source
javac -d bin -sourcepath src src/com/custom/filter/CustomHeaderFilter.java
 
## create Jar
jar cvfm CustomHeaderFilter.jar manifest.txt -C bin .
```
## 2. Copy the jar file in weblogic Domain's lib directory

``` bash
cp CustomHeaderFilter.jar /u01/oracle/user_projects/domains/jmstest/lib/
```
## 3. Add <filter> and  <filter-mapping> tags in web.xml


The init params starting with set.header. are considered for setting headers. the header name will be the string of param-name after 'set.header.' and header values will be taken from <param-value>


The init param dump.request and dump.response can be used with value 'true' to do custom logging. in the sample we are just doing system.out.println. it can be customised as per your need

``` xml
<filter>
    <filter-name>CustomHeaderFilter</filter-name>
    <filter-class>com.oracle.fsgbu.custom.filter.CustomHeaderFilter</filter-class>
    <init-param>                                                                                              
        <param-name>set.header.myCustomHeader</param-name>
        <param-value>customValue</param-value>
    </init-param>
    <init-param>
        <param-name>set.header.Content-Security-Policy</param-name>
        <param-value>default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self';</param-value>
    </init-param>
    <init-param>
        <param-name>dump.request</param-name>                                                                
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>dump.response</param-name>                                                                    
        <param-value>true</param-value>                                                                                    
    </init-param>
</filter>
 
<filter-mapping>
    <filter-name>CustomHeaderFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```


## Conclusion

WebLogic Server's filter feature offers a powerful mechanism for customizing request processing and enhancing monitoring capabilities without modifying the underlying code of a web application. By leveraging filters, developers and administrators can add custom HTTP headers, introduce additional logging, perform request/response manipulation, and more. This flexibility enables organizations to tailor their application behavior and monitoring to meet specific requirements, improving efficiency, maintainability, and troubleshooting capabilities.

In this article, we explored two common use cases of WebLogic filters: adding custom HTTP headers and enhancing logging for monitoring purposes. These examples illustrate the versatility of filters in extending the functionality of a web application. By harnessing the power of WebLogic's filter feature, organizations can achieve greater control and observability over their applications, ultimately delivering a more robust and efficient user experience.
