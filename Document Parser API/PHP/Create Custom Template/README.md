## How to create custom template for document parser API in PHP with PDF.co Web API PDF.co Web API is the flexible Web API that includes full set of functions from e-signature requests to data extraction, OCR, images recognition, pdf splitting and pdf splitting. Can also generate barcodes and read barcodes from images, scans and pdf.

## REQUEST FREE TECH SUPPORT

[Click here to get in touch](https://bytescout.zendesk.com/hc/en-us/requests/new?subject=PDF.co%20Web%20API%20Question)

or just send email to [support@bytescout.com](mailto:support@bytescout.com?subject=PDF.co%20Web%20API%20Question) 

## ON-PREMISE OFFLINE SDK 

[Get Your 60 Day Free Trial](https://bytescout.com/download/web-installer?utm_source=github-readme)
[Explore Documentation](https://bytescout.com/documentation/index.html?utm_source=github-readme)
[Explore Source Code Samples](https://github.com/bytescout/ByteScout-SDK-SourceCode/)
[Sign Up For Online Training](https://academy.bytescout.com/)


## ON-DEMAND REST WEB API

[Get your API key](https://app.pdf.co/signup?utm_source=github-readme)
[Security](https://pdf.co/security)
[Explore Web API Documentation](https://apidocs.pdf.co?utm_source=github-readme)
[Explore Web API Samples](https://github.com/bytescout/ByteScout-SDK-SourceCode/tree/master/PDF.co%20Web%20API)

## VIDEO REVIEW

[https://www.youtube.com/watch?v=NEwNs2b9YN8](https://www.youtube.com/watch?v=NEwNs2b9YN8)




<!-- code block begin -->

##### **SampleTemplate.yml:**
    
```
templateName: My Custom Template
templateVersion: 4
templatePriority: 0
detectionRules:
  keywords:
  - Your Company Name
  - Invoice No\.
  - TOTAL
objects:
- name: total
  objectType: field
  fieldProperties:
    fieldType: macros
    expression: TOTAL{{Spaces}}{{Number}}
    dataType: decimal
    pageIndex: 0
- name: dateIssued
  objectType: field
  fieldProperties:
    fieldType: macros
    expression: Invoice Date {{SmartDate}}
    dataType: date
    dateFormat: auto-mdy
    pageIndex: 0
- name: invoiceId
  objectType: field
  fieldProperties:
    fieldType: macros
    expression: Invoice No. {{Digits}}
    pageIndex: 0
- name: companyName
  objectType: field
  fieldProperties:
    fieldType: static
    expression: Vendor Company
    pageIndex: 0
- name: billTo
  objectType: field
  fieldProperties:
    fieldType: rectangle
    rectangle:
    - 32.25
    - 150
    - 348
    - 70.5
    pageIndex: 0
- name: notes
  objectType: field
  fieldProperties:
    fieldType: rectangle
    rectangle:
    - 32.25
    - 227.25
    - 531
    - 47.25
    pageIndex: 0
- name: table1
  objectType: table
  tableProperties:
    start:
      expression: Item\s+Quantity\s+Price\s+Total
      regex: true
    end:
      expression: TOTAL
      regex: true
    row:
      expression: ^\s*(?<description>\w+.*)(?<quantity>\d+)\s+(?<unitPrice>\d+\.\d{2})\s+(?<itemTotal>\d+\.\d{2})\s*$
      regex: true


```

<!-- code block end -->    

<!-- code block begin -->

##### **program.php:**
    
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document Parse Results</title>
</head>
<body>

<?php 


// Get submitted form data
$apiKey = $_POST["apiKey"]; // The authentication key (API Key). Get your own by registering at https://app.pdf.co


// 1. RETRIEVE THE PRESIGNED URL TO UPLOAD THE FILE.
// * If you already have the direct PDF file link, go to the step 3.

// Create URL
$url = "https://api.pdf.co/v1/file/upload/get-presigned-url" . 
    "?name=" . $_FILES["file"]["tmp_name"] .
    "&contenttype=application/octet-stream";
    
// Create request
$curl = curl_init();
curl_setopt($curl, CURLOPT_HTTPHEADER, array("x-api-key: " . $apiKey));
curl_setopt($curl, CURLOPT_URL, $url);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
// Execute request
$result = curl_exec($curl);

if (curl_errno($curl) == 0)
{
    $status_code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
    
    if ($status_code == 200)
    {
        $json = json_decode($result, true);
        
        // Get URL to use for the file upload
        $uploadFileUrl = $json["presignedUrl"];
        // Get URL of uploaded file to use with later API calls
        $uploadedFileUrl = $json["url"];
        
        // 2. UPLOAD THE FILE TO CLOUD.
        
        $localFile = $_FILES["fileInput"]["tmp_name"];
        $fileHandle = fopen($localFile, "r");
        
        curl_setopt($curl, CURLOPT_URL, $uploadFileUrl);
        curl_setopt($curl, CURLOPT_HTTPHEADER, array("content-type: application/octet-stream"));
        curl_setopt($curl, CURLOPT_PUT, true);
        curl_setopt($curl, CURLOPT_INFILE, $fileHandle);
        curl_setopt($curl, CURLOPT_INFILESIZE, filesize($localFile));

        // Execute request
        curl_exec($curl);
        
        fclose($fileHandle);
        
        if (curl_errno($curl) == 0)
        {
            $status_code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
            
            if ($status_code == 200)
            {
                // Read all template texts
                $templateText = file_get_contents($_FILES["fileTemplate"]["tmp_name"]);

                // 3. PARSE UPLOADED PDF DOCUMENT
                ParseDocument($apiKey, $uploadedFileUrl, $templateText);
            }
            else
            {
                // Display request error
                echo "<p>Status code: " . $status_code . "</p>"; 
                echo "<p>" . $result . "</p>"; 
            }
        }
        else
        {
            // Display CURL error
            echo "Error: " . curl_error($curl);
        }
    }
    else
    {
        // Display service reported error
        echo "<p>Status code: " . $status_code . "</p>"; 
        echo "<p>" . $result . "</p>"; 
    }
    
    curl_close($curl);
}
else
{
    // Display CURL error
    echo "Error: " . curl_error($curl);
}

function ParseDocument($apiKey, $uploadedFileUrl, $templateText) 
{
    // (!) Make asynchronous job
    $async = TRUE;

    // Prepare URL for Document parser API call.
    // See documentation: https://apidocs.pdf.co/?#1-pdfdocumentparser
    $url = "https://api.pdf.co/v1/pdf/documentparser";

    // Prepare requests params
    $parameters = array();
    $parameters["url"] = $uploadedFileUrl;
    $parameters["template"] = $templateText;
    $parameters["async"] = $async;

    // Create Json payload
    $data = json_encode($parameters);

    // Create request
    $curl = curl_init();
    curl_setopt($curl, CURLOPT_HTTPHEADER, array("x-api-key: " . $apiKey, "Content-type: application/json"));
    curl_setopt($curl, CURLOPT_URL, $url);
    curl_setopt($curl, CURLOPT_POST, true);
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    curl_setopt($curl, CURLOPT_POSTFIELDS, $data);

    // Execute request
    $result = curl_exec($curl);
    echo $result . "<br/>";

    if (curl_errno($curl) == 0)
    {
        $status_code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
        
        if ($status_code == 200)
        {
            $json = json_decode($result, true);
        
            if ($json["error"] == false)
            {
                // URL of generated JSON file that will available after the job completion
                $resultFileUrl = $json["url"];
                // Asynchronous job ID
                $jobId = $json["jobId"];
                
                // Check the job status in a loop
                do
                {
                    $status = CheckJobStatus($jobId, $apiKey); // Possible statuses: "working", "failed", "aborted", "success".
                    
                    // Display timestamp and status (for demo purposes)
                    echo "<p>" . date(DATE_RFC2822) . ": " . $status . "</p>";
                    
                    if ($status == "success")
                    {
                        // Display link to JSON file with information about parsed fields
                        echo "<div>## Parsing Result:<a href='" . $resultFileUrl . "' target='_blank'>" . $resultFileUrl . "</a></div>";
                        break;
                    }
                    else if ($status == "working")
                    {
                        // Pause for a few seconds
                        sleep(3);
                    }
                    else 
                    {
                        echo $status . "<br/>";
                        break;
                    }
                }
                while (true);
            }
            else
            {
                // Display service reported error
                echo "<p>Error: " . $json["message"] . "</p>"; 
            }
        }
        else
        {
            // Display request error
            echo "<p>Status code: " . $status_code . "</p>"; 
            echo "<p>" . $result . "</p>"; 
        }
    }
    else
    {
        // Display CURL error
        echo "Error: " . curl_error($curl);
    }
}

function CheckJobStatus($jobId, $apiKey)
{
    $status = null;
    
	// Create URL
    $url = "https://api.pdf.co/v1/job/check";
    
    // Prepare requests params
    $parameters = array();
    $parameters["jobid"] = $jobId;

    // Create Json payload
    $data = json_encode($parameters);

    // Create request
    $curl = curl_init();
    curl_setopt($curl, CURLOPT_HTTPHEADER, array("x-api-key: " . $apiKey, "Content-type: application/json"));
    curl_setopt($curl, CURLOPT_URL, $url);
    curl_setopt($curl, CURLOPT_POST, true);
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
    
    // Execute request
    $result = curl_exec($curl);
    
    if (curl_errno($curl) == 0)
    {
        $status_code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
        
        if ($status_code == 200)
        {
            $json = json_decode($result, true);
        
            if ($json["error"] == false)
            {
                $status = $json["status"];
            }
            else
            {
                // Display service reported error
                echo "<p>Error: " . $json["message"] . "</p>"; 
            }
        }
        else
        {
            // Display request error
            echo "<p>Status code: " . $status_code . "</p>"; 
            echo "<p>" . $result . "</p>"; 
        }
    }
    else
    {
        // Display CURL error
        echo "Error: " . curl_error($curl);
    }
    
    // Cleanup
    curl_close($curl);
    
    return $status;
}

?>

</body>
</html>
```

<!-- code block end -->