# Invoke<a name="API_Invoke"></a>

Invokes a Lambda function\. You can invoke a function synchronously \(and wait for the response\), or asynchronously\. To invoke a function asynchronously, set `InvocationType` to `Event`\.

For [synchronous invocation](https://docs.aws.amazon.com/lambda/latest/dg/invocation-sync.html), details about the function response, including errors, are included in the response body and headers\. For either invocation type, you can find more information in the [execution log](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-functions.html) and [trace](https://docs.aws.amazon.com/lambda/latest/dg/lambda-x-ray.html)\.

When an error occurs, your function may be invoked multiple times\. Retry behavior varies by error type, client, event source, and invocation type\. For example, if you invoke a function asynchronously and it returns an error, Lambda executes the function up to two more times\. For more information, see [Retry Behavior](https://docs.aws.amazon.com/lambda/latest/dg/retries-on-errors.html)\.

For [asynchronous invocation](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html), Lambda adds events to a queue before sending them to your function\. If your function does not have enough capacity to keep up with the queue, events may be lost\. Occasionally, your function may receive the same event multiple times, even if no error occurs\. To retain events that were not processed, configure your function with a [dead\-letter queue](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html#dlq)\.

The status code in the API response doesn't reflect function errors\. Error codes are reserved for errors that prevent your function from executing, such as permissions errors, [limit errors](https://docs.aws.amazon.com/lambda/latest/dg/limits.html), or issues with your function's code and configuration\. For example, Lambda returns `TooManyRequestsException` if executing the function would cause you to exceed a concurrency limit at either the account level \(`ConcurrentInvocationLimitExceeded`\) or function level \(`ReservedFunctionConcurrentInvocationLimitExceeded`\)\.

For functions with a long timeout, your client might be disconnected during synchronous invocation while it waits for a response\. Configure your HTTP client, SDK, firewall, proxy, or operating system to allow for long connections with timeout or keep\-alive settings\.

This operation requires permission for the `lambda:InvokeFunction` action\.

## Request Syntax<a name="API_Invoke_RequestSyntax"></a>

```
POST /2015-03-31/functions/FunctionName/invocations?Qualifier=Qualifier HTTP/1.1
X-Amz-Invocation-Type: InvocationType
X-Amz-Log-Type: LogType
X-Amz-Client-Context: ClientContext

Payload
```

## URI Request Parameters<a name="API_Invoke_RequestParameters"></a>

The request requires the following URI parameters\.

 ** [ClientContext](#API_Invoke_RequestSyntax) **   <a name="SSS-Invoke-request-ClientContext"></a>
Up to 3583 bytes of base64\-encoded data about the invoking client to pass to the function in the context object\.

 ** [FunctionName](#API_Invoke_RequestSyntax) **   <a name="SSS-Invoke-request-FunctionName"></a>
The name of the Lambda function, version, or alias\.  

**Name formats**
+  **Function name** \- `my-function` \(name\-only\), `my-function:v1` \(with alias\)\.
+  **Function ARN** \- `arn:aws:lambda:us-west-2:123456789012:function:my-function`\.
+  **Partial ARN** \- `123456789012:function:my-function`\.
You can append a version number or alias to any of the formats\. The length constraint applies only to the full ARN\. If you specify only the function name, it is limited to 64 characters in length\.  
Length Constraints: Minimum length of 1\. Maximum length of 170\.  
Pattern: `(arn:(aws[a-zA-Z-]*)?:lambda:)?([a-z]{2}(-gov)?-[a-z]+-\d{1}:)?(\d{12}:)?(function:)?([a-zA-Z0-9-_\.]+)(:(\$LATEST|[a-zA-Z0-9-_]+))?` 

 ** [InvocationType](#API_Invoke_RequestSyntax) **   <a name="SSS-Invoke-request-InvocationType"></a>
Choose from the following options\.  
+  `RequestResponse` \(default\) \- Invoke the function synchronously\. Keep the connection open until the function returns a response or times out\. The API response includes the function response and additional data\.
+  `Event` \- Invoke the function asynchronously\. Send events that fail multiple times to the function's dead\-letter queue \(if it's configured\)\. The API response only includes a status code\.
+  `DryRun` \- Validate parameter values and verify that the user or role has permission to invoke the function\.
Valid Values:` Event | RequestResponse | DryRun` 

 ** [LogType](#API_Invoke_RequestSyntax) **   <a name="SSS-Invoke-request-LogType"></a>
Set to `Tail` to include the execution log in the response\.  
Valid Values:` None | Tail` 

 ** [Qualifier](#API_Invoke_RequestSyntax) **   <a name="SSS-Invoke-request-Qualifier"></a>
Specify a version or alias to invoke a published version of the function\.  
Length Constraints: Minimum length of 1\. Maximum length of 128\.  
Pattern: `(|[a-zA-Z0-9$_-]+)` 

## Request Body<a name="API_Invoke_RequestBody"></a>

The request accepts the following binary data\.

 ** [Payload](#API_Invoke_RequestSyntax) **   <a name="SSS-Invoke-request-Payload"></a>
The JSON that you want to provide to your Lambda function as input\.

## Response Syntax<a name="API_Invoke_ResponseSyntax"></a>

```
HTTP/1.1 StatusCode
X-Amz-Function-Error: FunctionError
X-Amz-Log-Result: LogResult
X-Amz-Executed-Version: ExecutedVersion

Payload
```

## Response Elements<a name="API_Invoke_ResponseElements"></a>

If the action is successful, the service sends back the following HTTP response\.

 ** [StatusCode](#API_Invoke_ResponseSyntax) **   <a name="SSS-Invoke-response-StatusCode"></a>
The HTTP status code is in the 200 range for a successful request\. For the `RequestResponse` invocation type, this status code is 200\. For the `Event` invocation type, this status code is 202\. For the `DryRun` invocation type, the status code is 204\.

The response returns the following HTTP headers\.

 ** [ExecutedVersion](#API_Invoke_ResponseSyntax) **   <a name="SSS-Invoke-response-ExecutedVersion"></a>
The version of the function that executed\. When you invoke a function with an alias, this indicates which version the alias resolved to\.  
Length Constraints: Minimum length of 1\. Maximum length of 1024\.  
Pattern: `(\$LATEST|[0-9]+)` 

 ** [FunctionError](#API_Invoke_ResponseSyntax) **   <a name="SSS-Invoke-response-FunctionError"></a>
If present, indicates that an error occurred during function execution\. Details about the error are included in the response payload\.  
+  `Handled` \- The runtime caught an error thrown by the function and formatted it into a JSON document\.
+  `Unhandled` \- The runtime didn't handle the error\. For example, the function ran out of memory or timed out\.

 ** [LogResult](#API_Invoke_ResponseSyntax) **   <a name="SSS-Invoke-response-LogResult"></a>
The last 4 KB of the execution log, which is base64 encoded\.

The response returns the following as the HTTP body\.

 ** [Payload](#API_Invoke_ResponseSyntax) **   <a name="SSS-Invoke-response-Payload"></a>
The response from the function, or an error object\.

## Errors<a name="API_Invoke_Errors"></a>

 **EC2AccessDeniedException**   
Need additional permissions to configure VPC settings\.  
HTTP Status Code: 502

 **EC2ThrottledException**   
AWS Lambda was throttled by Amazon EC2 during Lambda function initialization using the execution role provided for the Lambda function\.  
HTTP Status Code: 502

 **EC2UnexpectedException**   
AWS Lambda received an unexpected EC2 client exception while setting up for the Lambda function\.  
HTTP Status Code: 502

 **ENILimitReachedException**   
AWS Lambda was not able to create an Elastic Network Interface \(ENI\) in the VPC, specified as part of Lambda function configuration, because the limit for network interfaces has been reached\.  
HTTP Status Code: 502

 **InvalidParameterValueException**   
One of the parameters in the request is invalid\. For example, if you provided an IAM role for AWS Lambda to assume in the `CreateFunction` or the `UpdateFunctionConfiguration` API, that AWS Lambda is unable to assume you will get this exception\.  
HTTP Status Code: 400

 **InvalidRequestContentException**   
The request body could not be parsed as JSON\.  
HTTP Status Code: 400

 **InvalidRuntimeException**   
The runtime or runtime version specified is not supported\.  
HTTP Status Code: 502

 **InvalidSecurityGroupIDException**   
The Security Group ID provided in the Lambda function VPC configuration is invalid\.  
HTTP Status Code: 502

 **InvalidSubnetIDException**   
The Subnet ID provided in the Lambda function VPC configuration is invalid\.  
HTTP Status Code: 502

 **InvalidZipFileException**   
AWS Lambda could not unzip the deployment package\.  
HTTP Status Code: 502

 **KMSAccessDeniedException**   
Lambda was unable to decrypt the environment variables because KMS access was denied\. Check the Lambda function's KMS permissions\.  
HTTP Status Code: 502

 **KMSDisabledException**   
Lambda was unable to decrypt the environment variables because the KMS key used is disabled\. Check the Lambda function's KMS key settings\.  
HTTP Status Code: 502

 **KMSInvalidStateException**   
Lambda was unable to decrypt the environment variables because the KMS key used is in an invalid state for Decrypt\. Check the function's KMS key settings\.  
HTTP Status Code: 502

 **KMSNotFoundException**   
Lambda was unable to decrypt the environment variables because the KMS key was not found\. Check the function's KMS key settings\.   
HTTP Status Code: 502

 **RequestTooLargeException**   
The request payload exceeded the `Invoke` request body JSON input limit\. For more information, see [Limits](https://docs.aws.amazon.com/lambda/latest/dg/limits.html)\.   
HTTP Status Code: 413

 **ResourceNotFoundException**   
The resource \(for example, a Lambda function or access policy statement\) specified in the request does not exist\.  
HTTP Status Code: 404

 **ServiceException**   
The AWS Lambda service encountered an internal error\.  
HTTP Status Code: 500

 **SubnetIPAddressLimitReachedException**   
AWS Lambda was not able to set up VPC access for the Lambda function because one or more configured subnets has no available IP addresses\.  
HTTP Status Code: 502

 **TooManyRequestsException**   
Request throughput limit exceeded\.  
HTTP Status Code: 429

 **UnsupportedMediaTypeException**   
The content type of the `Invoke` request body is not JSON\.  
HTTP Status Code: 415

## See Also<a name="API_Invoke_SeeAlso"></a>

For more information about using this API in one of the language\-specific AWS SDKs, see the following:
+  [AWS Command Line Interface](https://docs.aws.amazon.com/goto/aws-cli/lambda-2015-03-31/Invoke) 
+  [AWS SDK for \.NET](https://docs.aws.amazon.com/goto/DotNetSDKV3/lambda-2015-03-31/Invoke) 
+  [AWS SDK for C\+\+](https://docs.aws.amazon.com/goto/SdkForCpp/lambda-2015-03-31/Invoke) 
+  [AWS SDK for Go](https://docs.aws.amazon.com/goto/SdkForGoV1/lambda-2015-03-31/Invoke) 
+  [AWS SDK for Go \- Pilot](https://docs.aws.amazon.com/goto/SdkForGoPilot/lambda-2015-03-31/Invoke) 
+  [AWS SDK for Java](https://docs.aws.amazon.com/goto/SdkForJava/lambda-2015-03-31/Invoke) 
+  [AWS SDK for JavaScript](https://docs.aws.amazon.com/goto/AWSJavaScriptSDK/lambda-2015-03-31/Invoke) 
+  [AWS SDK for PHP V3](https://docs.aws.amazon.com/goto/SdkForPHPV3/lambda-2015-03-31/Invoke) 
+  [AWS SDK for Python](https://docs.aws.amazon.com/goto/boto3/lambda-2015-03-31/Invoke) 
+  [AWS SDK for Ruby V2](https://docs.aws.amazon.com/goto/SdkForRubyV2/lambda-2015-03-31/Invoke) 