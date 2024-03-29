# ProofTag-Challenge

## Introduction:

The prooftag authentication web service verifies the the authenticity of a product with a seal that composes of different components.
- 2D code (QR or Datamatrix)
- A visual control element (BubbleTag, FiberTag, Ramdot)
- Alphanumeric reference (Tag number) | What will be used to query Prooftag Web Services

Request for additional data has limited noa.
If error occurs on the control data, seal is blocked for a spex period(default 24 hours)

## Notes:

### Authenticity Web Service

Authenticity Service is a SOAP web service that provides all information related to a seal.
Web Service is accessible via URL: https://ws.prooftag.com/AuthenticityService.asmx 

### GetSealData()

GetSealData function allows us to obtain:
- seal status
- image of visual control element
- cert.
    - activation date
    - validity date 
- prod.
    - name
    - description
    - image
- id of container

- Exchange is done through XML files. (params of req. will be def. in an XMlIn file  and res. will be in XMlOut)
- Only returns what was req. in XMlIn file

Requirements:
- client identifier(APP_ID) | attached onto customer account in ProofTag
    - default APP_ID: "TEST". can only use query ref. of testing and dev.
    - in prod. APP_ID: "PRODUCTION" allowing it to query real ref.
- params. in XMlIn file must contain `<requestedDataDetail>` tags describing a setting or option for GetSealData func.
    - three required params:
        - ref. # of seal
        - customer ID(APP_ID) 
        - IP of user of the website
    - optional tags define what data is required and what will be returned by WebService
    - must be encapsulated in `<request>` and `<requestData>` tags
    - example:
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
        <request>
            <requestedData>
                <requestedDataDetail>
                    <dataType>TAG_NUMBER</dataType>
                    <dataValue>0PA00AAAB46602</dataValue>
                </requestedDataDetail>
                <requestedDataDetail>
                    <dataType>GET_BUBBLE</dataType>
                </requestedDataDetail>
            </requestedData>
        </request>
    ```
    - forEach `<requestedDataDetail>` tag must contain a tag describing the req. param. type(dataType). 
    - According to dataType additional tags can be added
        - dataType | Type of param
        - dataValue | Value of param
        - Required according to data type used
            - dataId | ID of the data.
            - dataContent | Base64 format image. 
            - dataLabel | Wording
        - dataName | Parameter Name
    - Parameter type for dataType:
        - PARAM. Types | Description | Complement Tags
        - TAG_NUMBER   | Seal Reference   | `<dataValue>`: [alphanumeric] Value of the reference. The parameter is not case-sensitive
        - GET_BUBBLE   | Request image of security element | 
        - GET_PRODUCT  | Request product imaage and description | 
        - GET_CONTAINER| Request identifier of the container |
        - GET_DATE_ACTIVATION | Request activation date of the certificate |
        - GET_DATE_VALIDITY | Request validity date of the certificate | 
        - LANG         | The language the info will be returned in |`<dataValue>` [ISO code]: Code of the requested language for multilingual fields. 
                                                                        The LANG parameter value must comply with the ISO 639 1 code 
                                                                        (see table http://fr.wikipedia.org/wiki/Liste_des_codes_ISO_639-1 ), two lowercase letters.
        - BUBBLE_WIDTH | Max width of seal image returned by web service. Can only reduce |`<dataValue>`: [numerical] Size in pixels. The width / height ratio of the image is                                                                                       preserved.
        - BUBBLE_HEIGHT | Max height of seal image returned by web service. Can only reduce | ^^
        - PRODUCT_WIDTH | Max width of product image returned by web service. Can only reduce | ^^
        - PRODUCT_HEIGHT | Max height of product iamge returned by web service. Can only reduce | ^^
        - APP_ID       | Unique identifier of the calling application. Defines certs. accessible by app | `<dataValue>` : [alphanumeric] Value of the identifier. This                                                                                                            identifier is generated by Prooftag and communicated to partner                                                                                                         applications.
        - SESSION_ID   | Unique identified of the calling session. Mandatory in the case of a loop call to web service. If only one call then not needed | `<dataValue>`:           [alphanumeric] ID value. This identifier is returned during the first call to the web service. 
        - CLIENT_IP    | IP address of the site client. Used for traceability | `<dataValue>`[alphanumeric] value of the IP
        - CONTROL_KEY  | Desribe control data's access to protected access certs. | `<dataId>`: Identifier of the control value. Since a certificate can be protected by several control values, there can be several requestedDataDetail of type CONTROL_KEY, the dataId allows their differentiation. The dataId tag must be provided in the response.
        `<dataValue>` : tag with the value of the control data to be provided to the web service in return. `<dataLabel>` : label of the control data for displaying a form.
        - CONTROL_BUBBLE | All the CONTROL_BUBBLE tags define a list of of Bubble Tag images which the user must select the ref. img. in the process | `<dataValue>`: The        picture’s index of the Bubble Tag ™ in the checklist. `<dataContent>` : base code 64 of the Bubble Tag ™ image
    - Example of XMlIn file(requires three param. TAG_NUMBER, APP_ID, CLIENT_IP w/ optional GET_BUBBLE):
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <request>
            <requestedData>
                <requestedDataDetail>
                    <dataType>TAG_NUMBER</dataType>
                    <dataValue>0PA00AAAB46602</dataValue>
                </requestedDataDetail>
                <requestedDataDetail>
                    <dataType>APP_ID</dataType>
                    <dataValue>01XXX01-XX02-4XXX-XX1A-334B0DA4XXXX</dataValue>
                </requestedDataDetail>
                <requestedDataDetail>
                    <dataType>CLIENT_IP</dataType>
                    <dataValue>255.255.255.255</dataValue>
                </requestedDataDetail>
                <requestedDataDetail>
                    <dataType>GET_BUBBLE</dataType>
                </requestedDataDetail>
            </requestedData>
        </request>
        ```

Call GetSealData:
- done through SOAP call(XML-based protocol for accessing web services over HTTP). 
    - uses two steps:
        - initialization of SOAP client w/ URL of web service to call
        - calling GetSealData function w/ XMlIn file param

### Web Service Response

Response is done through XMlOut file and returned by GetSealData function will always contain a `<response>` w/ the following tags:
- Status | indicates status of exchange w/ web service values:
    - END | exchange is finished. the message, msg_id, tag, and/or product tags contain response of web service.
        - LOOP | exchange is not completed. web service is awaiting for second call. `<dataRequestDetail>` describe additional info. to be provided
        - ERROR | exchange ended w/ error. msg_id and error tags contain info about the error
    - MSG_ID | Return code
    - MESSAGE | Return message
    - If error occurs an ERROR tag is also sent w/ error description in FR.
        - MESSAGE and ERROR always returned in FR
        - To handle in diff. language use msg_id tag to display custom message

Messages and Return Codes:
- MSG_ID    | Message code  | Message
- 0         | Ok            | Ok. No error
- 1         | ERR_UNKNOWN   | Unknown Error
- 10         | ERR_INVALID_XML_IN    | XMlIn param is not correct format
- 11        | ERR_INVALID_XML_REQUEST | XMlIn param does not contain valid `<request>` tag
- 20        | ERR_SESSION   | The seesion has expired since the last call
- 30         | ERR_BLOCKED_SEAL | The certificate is blocked following a call with wrong control values. Important: On the test references, the blocking is not effective but the web service still returns the message ERR_BLOCKED_SEAL.
- 100       | MSG_UNKNOWN_REFERENCE | The ref. is unknown
- 101       | MSG_MISSING_REFERENCE | The ref. is not defined in the query
- 106       | MSG_NEED_CONTROL_KEYS | The control data values are needed to access this ref.
- 110       | MSG_EXPIRED_REFERENCE | The cert. corresponding to ref. expired

If GET_PRODUCT was req. then web service will return info in `<tag>`. If no GET_BUBBLE, GET_DATE_ACTIVATION, or  GET_DATE_VALIDITY params. defined then will return img. of security element by default.
- Tag   | XMlIn necessaire  | Description
- tag_number |              | Seal Reference
- tag_status |              | 1(activated) or 2(archived)
- Tag_image_64 | GET_BUBBLE | Image(base64 code) of visual control element
- tag_date_activation | GET_DATE_ACTIVATION | Certification activation date in yyyy-mm-dd format
- tag_date_validity | GET_DATE_VALIDITY | Date of validity of cert. in yyyy-mm-dd format. If tag is empty, cert. still valid
- target_country |          | ISO code of the target country of the cert. The target country is returned if defined when cert. was activated

If GET_CONTAINER param. was req. the web service will return info in `<container>`, given that cert is a container and/or the cert. is in a container
- Tag   | Description
- is_container | Cert. is a container if value returned is 1. GetContainerInfo method provides container specific info.
- parent_container_ref | Parent container cert. ref.
- parent_container_id  | Parent container cert. id

If GET_PRODUCT param. was req. and a product has been associated with cert., the WS will return info in `<product>`
- Tag   | Desc.
- product_number | Product ref. (Don't display)
- product_name   | Product name
- product_desc   | Desc. of product sheet
- product_image_64 | Product image(base64 code). Maybe multiple `<product_image_64>` for multipage prod.
- extern_id      | Ext. product identifier in the system. Tag is present only if ext. identifier is defined
- CODE GTIN

If web service requires additional info., XMlOut file will contain `<dataRequestDetail>` containing details of required info. Type of info is defined by `<dataType>`, TAG_NUMBER, SESSION_ID, COTROL_KEY, or CONTROL_BUBBLE
- TAG   | Desc.
- dataType | Type of missing data to provide in req.
- dataName | Name of data(used for TAG_NUMBER or CONTROL_KEY)
- dataValue | Value data(used for SESSION_ID or CONTROL_BUBBLE)
- dataContent | Image(base64 code) of visual control element(used for CONTROL_BUBBLE)
- dataLabel | Label for disp. data
- dataComment | Explanatory comment

Examples of Web service returns:
- success: 
```xml
        <?xml version="1.0" encoding="UTF-8"?>
            <response>
                <status>END</status>
                <message>OK</message>
                <tag>
                    <tag_number>P00A00B000001</tag_number>
                    <tag_status>active</tag_status>
                    <tag_date_activation>2006-10-05</tag_date_activation>
                    <tag_date_validity></tag_date_validity>
                    <tag_image_64>/9j/…..==</tag_image_64>
                    <target_country>DE</target_country>
                </tag>
            </response>
```
- failure:
```xml
        <?xml version="1.0" encoding="UTF-8"?>
            <response>
                <status>ERROR</status>
                <msg_id>100</msg_id>
            </response>
```
- error:
```xml
        <?xml version="1.0" encoding="UTF-8"?>
            <response>
                <status>ERROR</status>
                    <message>Une erreur est survenue.</message>
                    <msg_id>10</msg_id>
                    <error>Le paramètre XmlIn n'est pas du XML valide.</error>
            </response>
```

### Control Data Request

If access to cert. is protected by one or more control values, the WS will wait 20 minutes for a second call containing requested values and a session code. If wrong control values were provided and/or number of attempts exceeded(assuming time alloted has past) then cert. is blocked for a period of time(default 24 hours)

When requesting additional control data the WS returns LOOP status and describes values needed in `<dataRequestDetail>` which contains `<dataType>` of CONTROL_BUBBLE or CONTROL_KEY

Control key: (control by text value)
- maybe CCP code on seal or any other textual data to authenticate product.(Only one attempt)
- XMlOut file example:
```xml
<?xml version="1.0" encoding="UTF-8"?>
    <response>
        <status>LOOP</status>
        <message>Veuillez saisir les données de contrôle pour visualiser le certificat.</message>
        <msg_id>106</msg_id>
        <dataRequest>
            <dataRequestDetail>
                <dataType>CONTROL_KEY</dataType>
                <dataId>123</dataId>
                <dataLabel><![CDATA[Numéro de référence]]></dataLabel>
            </dataRequestDetail>
        </dataRequest>
        <dataRequest>
            <dataRequestDetail>
                <dataType>SESSION_ID</dataType>
                <dataId>xxxx1xxxxz1o0f45xpteia55</dataId>
            </dataRequestDetail>
        </dataRequest>
    </response>
```
- Second call must then include `<dataType>CONTROL_KEY</dataType>`, `<dataId>123</dataId>`, and `<dataValue>(Whatever is required)</dataValue>`
```xml
<requestedDataDetail>
    <dataType>CONTROL_KEY</dataType>
    <dataId>123</dataId>
    <dataValue><![CDATA[user input]]></dataValue>
    </requestedDataDetail>
```

Control Bubble: (Image Captcha Bubble Control)
- The WS will return a series of images, one of which corresponds to the bubble code on the seal.
- Number of captcha bubbles sent depends on customer's configs. Determines # of attemps
    - Bubble Captcha #  | Authorized attemps #
    - 3                 | 1
    - 6                 | 2
    - 9                 | 3
- Example of XMlOut file:
```xml
    <?xml version="1.0" encoding="UTF-8"?>
        <response>
            <status>LOOP</status>
            <message> Please enter the control data to see the certificate. </message>
            <msg_id>106</msg_id>
            <dataRequest>
                <dataRequestDetail>
                    <dataType>CONTROL_ BUBBLE </dataType>
                    <dataValue>1</dataValue >
                    <dataContent>base64 image 1….. </dataContent >
                </dataRequestDetail>
                <dataRequestDetail>
                    <dataType>CONTROL_ BUBBLE </dataType>
                    <dataValue>2</dataValue >
                    <dataContent> base64 image 2….. </dataContent >
                </dataRequestDetail>
                <dataRequestDetail>
                    <dataType>CONTROL_ BUBBLE </dataType>
                    <dataValue>3</dataValue >
                    <dataContent> base64 image 3….. </dataContent >
                </dataRequestDetail>
            </dataRequest>
            <dataRequest>
                <dataRequestDetail>
                    <dataType>SESSION_ID</dataType>
                    <dataId>xxxx1xxxxz1o0f45xpteia55</dataId>
                </dataRequestDetail>
            </dataRequest>
        </response>
```
- For the second call to return user's selected image refer to image's <dataValue>
```xml
<requestedDataDetail>
    <dataType>CONTROL_BUBBLE</dataType>
    <dataValue> Index of the image selected by the user </dataValue>
    </requestedDataDetail>
```
    
For both control methods it's required to return SESSION_ID provided by the web service