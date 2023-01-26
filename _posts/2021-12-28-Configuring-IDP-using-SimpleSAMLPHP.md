**SimpleSAMLPHP** is an opensource project written in native PHP that deals with authentication. A docker image of SimpleSAMLPHP is built and kept in FSGBU JAPAC Consulting Docker registry that one can directly pull and run an instance of IDP for development and testing purpose


    Create a file saml20-sp-remote.php that has URLs with respective metadata. Refer below example.


    Note: The 'TAG' in the $metadata['TAG'] is the SAML Audience that we will be using in application. the URL is the Application URL that you want to configure in this IDP

``` 
        <?php
        /**
         * SAML 2.0 remote SP metadata for SimpleSAMLphp.
         *
         * See: https://simplesamlphp.org/docs/stable/simplesamlphp-reference-sp-remote
         */
          
        $metadata[getenv('SIMPLESAMLPHP_SP_ENTITY_ID')] = array(
            'AssertionConsumerService' => getenv('SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE'),
            'SingleLogoutService' => getenv('SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE'),
        );
        $metadata['fcubs'] = array(
            'AssertionConsumerService' => 'https://whf00bds:5002/FCJNeoWeb/LoginServlet?entity=ENTITY_ID1',
        );
        $metadata['obpay'] = array(
            'AssertionConsumerService' => 'https://whf00bds.in.oracle.com:5002/FCJNeoWeb/LoginServlet?entity=OBPAY',
        );
        $metadata['elcm'] = array(
            'AssertionConsumerService' => 'https://whf00bds.in.oracle.com:5002/FCJNeoWeb/LoginServlet?entity=ELCM',
        );
```

    Create file authsources.php with user credentails... Refer below example

```
        <?php
          
        $config = array(
          
            'admin' => array(
                'core:AdminPassword',
            ),
          
            'example-userpass' => array(
                'exampleauth:UserPass',
                'user1:user1pass' => array(
                    'uid' => array('1'),
                    'eduPersonAffiliation' => array('group1'),
                    'email' => 'user1@example.com',
                ),
                'user2:user2pass' => array(
                    'uid' => array('2'),
                    'eduPersonAffiliation' => array('group1'),
                    'email' => 'user2@example.com',
                ),
                'user3:user3pass' => array(
                    'uid' => array('3'),
                    'eduPersonAffiliation' => array('group1'),
                    'email' => 'user3@example.com',
                ),
            ),
          
        );

```

        Another example of authsources.php (for reference only)
         Expand source

    Run docker run command. Above 2 files can be mounted as volumes.

```
        docker run -dit --name=simple-saml-php \
        -p 8182:8080 \
        -v /scratch/share/saml/php/saml20-sp-remote.php:/var/www/simplesamlphp/metadata/saml20-sp-remote.php \
        -v /scratch/share/saml/php/authsources.php:/var/www/simplesamlphp/config/authsources.php \
        fsgbu-mum-128.snbomprshared1.gbucdsint02bom.oraclevcn.com:5000/utils/simple-saml-php:oracle-theme

```

Now an IDP is setup, and can be accessed using the URL http://<host>:8182/simplesaml/ To test the login go to  http://whf00ova.in.oracle.com:8080/simplesaml/module.php/core/authenticate.php 
    Public key of the IDP is available here http://<host>:8182/simplesaml/saml2/idp/metadata.php?output=xhtml
        IDP metadata - E.g. http://whf00ova.in.oracle.com:8080/simplesaml/saml2/idp/metadata.php?output=xhtml
        copy the value of tag <ds:X509Certificate> into a file.
        E.g.

```
-----BEGIN CERTIFICATE-----
MIIDXTCCAkWgAwIBAgIJALmVVuDWu4NYMA0GCSqGSIb3DQEBCwUAMEUxCzAJBgNVBAYTAkFVMRMwEQYDVQQ
IDApTb21lLVN0YXRlMSEwHwYDVQQKDBhJbnRlcm5ldCBXaWRnaXRzIFB0eSBMdGQwHhcNMTYxMjMxMTQzND
Q3WhcNNDgwNjI1MTQzNDQ3WjBFMQswCQYDVQQGEwJBVTETMBEGA1UECAwKU29tZS1TdGF0ZTEhMB8GA1UEC
gwYSW50ZXJuZXQgV2lkZ2l0cyBQdHkgTHRkMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzUCF
ozgNb1h1M0jzNRSCjhOBnR+uVbVpaWfXYIR+AhWDdEe5ryY+CgavOg8bfLybyzFdehlYdDRgkedEB/GjG8a
Jw06l0qF4jDOAw0kEygWCu2mcH7XOxRt+YAH3TVHa/Hu1W3WjzkobqqqLQ8gkKWWM27fOgAZ6GieaJBN6VB
SMMcPey3HWLBmc+TYJmv1dbaO2jHhKh8pfKw0W12VM8P1PIO8gv4Phu/uuJYieBWKixBEyy0lHjyixYFCR1
2xdh4CA47q958ZRGnnDUGFVE1QhgRacJCOZ9bd5t9mr8KLaVBYTCJo5ERE8jymab5dPqe5qKfJsCZiqWglb
jUo9twIDAQABo1AwTjAdBgNVHQ4EFgQUxpuwcs/CYQOyui+r1G+3KxBNhxkwHwYDVR0jBBgwFoAUxpuwcs/
CYQOyui+r1G+3KxBNhxkwDAYDVR0TBAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAQEAAiWUKs/2x/viNCKi3Y
6blEuCtAGhzOOZ9EjrvJ8+COH3Rag3tVBWrcBZ3/uhhPq5gy9lqw4OkvEws99/5jFsX1FJ6MKBgqfuy7yh5
s1YfM0ANHYczMmYpZeAcQf2CGAaVfwTTfSlzNLsF2lW/ly7yapFzlYSJLGoVE+OHEu8g5SlNACUEfkXw+5E
ghh+KzlIN7R6Q7r2ixWNFBC/jWf7NKUfJyX8qIG5md1YUeT6GBW9Bm2/1/RiO24JTaYlfLdKK9TYb8sG5B+
OLab2DImG99CJ25RkAcSobWNF5zD0O6lgOo3cEdB/ksCq3hmtlC/DlLZ/D8CJ+7VuZnS1rR2naQ==
-----END CERTIFICATE-----
 ```
