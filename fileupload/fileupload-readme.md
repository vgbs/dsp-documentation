---
title: Vaillant Group DSP Fileupload Help
description: File Upload Getting Started
---

[&larr; back to Overview](/fileupload)

## How does it work?

DSP fileupload is a REST API that allows you to upload files from your client application to Vaillant Group Salesforce
objects.
Before it is uploaded, a filecheck and malware analysis is performed. The file is then linked to the object of a record
id you provide as a parameter to your request.

Multiple files in one multipart request are supported. In the response, you will find various information about the
status of the file(s) in your request. See the [API documentation](fileupload-documentation.html) for more details.

If any error occurs during Salesforce upload, the file operation will be cancelled and the response will contain an
error message.

## Who can use it?

The DSP fileupload can be used by any team or agency developing a client, that uses Vaillant Group related services with
access to our Salesforce system.
Refer to the [service desk](https://service.dsp.vaillant-group.com), if you need to use fileupload functionality in your
application or you want to switch from an existing solution to our API.

## Authentication

The API is secured with an API key. The key is passed as a query parameter "`code`" in the request URL.

## What is the max. file size?

The file size limit is 30 MB per file provided in a multipart body.

## How do we check file properties?

The endpoint takes only multipart/form-data requests. The request body needs to contain at least one part.

| _Property_              | _Constraint_                                             |
|-------------------------|----------------------------------------------------------|
| **content-type header** | only _multipart/form-data_ allowed                       |
| **request body size**   | greater than 0 bytes                                     |
| **part header**         | Content-Disposition needs to contain `filename` property |
| **file extensions**     | no constraints                                           |
| **file mime type**      | no constraints                                           |
| **file size**           | greater than 0 bytes (30MB size limit)                   |

## Which Salesforce Objects are allowed?

The following Salesforce Objects are allowed:

* Contact
* Account
* Campaign
* Campaign Member
* Lead
* Opportunity
* Contact Brand Detail
* Account Brand Detail

## Example Code Snippet (Java)

```xml

<dependencies>
    ...
    <dependency>
        <groupId>org.apache.httpcomponents.client5</groupId>
        <artifactId>httpclient5</artifactId>
        <version>5.2.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.httpcomponents.core5</groupId>
        <artifactId>httpcore5</artifactId>
        <version>5.2.1</version>
    </dependency>
</dependencies>
```

```java
import org.apache.hc.client5.http.classic.methods.HttpPost;
import org.apache.hc.client5.http.entity.mime.FileBody;
import org.apache.hc.client5.http.entity.mime.FormBodyPartBuilder;
import org.apache.hc.client5.http.entity.mime.MultipartEntityBuilder;
import org.apache.hc.client5.http.impl.classic.CloseableHttpClient;
import org.apache.hc.client5.http.impl.classic.HttpClients;
import org.apache.hc.core5.http.ContentType;
import org.apache.hc.core5.http.HttpEntity;

import java.io.BufferedReader;
import java.io.File;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.stream.Collectors;

import static org.apache.hc.core5.http.HttpHeaders.CONTENT_DISPOSITION;

public class SampleFileUpload {
    private static final String baseUrl = "https://dsp-fileupload-qa.azurewebsites.net/api/files"; // <-- this is QA, for Prod just omit '-qa' from the hostname
    private static final String uploadUrl = baseUrl + "/upload?code=%s&recordID=%s";
    private static final String apiToken = "<apiToken>";
    private static final String sampleRecordId = "0013L11000V4pxv";


    private static final String boundary = "------myMultipartBoundary"; // this can be an arbitrarily chosen string which just needs to be unique enough to not occur within the body


    public static void main(String[] args) throws IOException {
        // Get the file to upload
        File file = new File("myfile.txt");

        // Create a connection to the server
        int responseCode;
        String responseBody;

        try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
            // Create a multipart entity
            MultipartEntityBuilder builder = MultipartEntityBuilder.create();
            builder.setBoundary(boundary);
            var body = new FileBody(file, ContentType.APPLICATION_OCTET_STREAM);
            var bodyPartBuilder = FormBodyPartBuilder.create();
            bodyPartBuilder
                    .setName("myfile name")
                    .setField("filename", file.getName())
                    .setBody(body);
            builder.addPart(bodyPartBuilder.build());

            HttpEntity entity = builder.build();

            // Create a POST request
            HttpPost request = new HttpPost(String.format(uploadUrl, apiToken, sampleRecordId));
            request.setEntity(entity);

            // Execute the request
            var response = httpClient.execute(request);

            // Get the response Body
            responseCode = response.getCode();
            responseBody = new BufferedReader(new InputStreamReader(response.getEntity().getContent()))
                    .lines()
                    .collect(Collectors.joining("\n"));
        }

        // Print the response
        System.out.println("Response Code: " + responseCode);
        System.out.println("Body: " + responseBody);

    }
}


```

