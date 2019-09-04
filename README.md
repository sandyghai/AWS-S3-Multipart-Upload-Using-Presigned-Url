AWS S3 Multipart Upload Using Presigned Url
===========================================

Amazon S3 offers the following options:

* Upload objects in a single operation—With a single PUT operation, you can upload objects up to 5 GB in size.
* Upload objects in parts—Using the multipart upload API, you can upload large objects, up to 5 TB.

The multipart upload API is designed to improve the upload experience for larger objects. You can upload objects in parts. These object parts can be uploaded independently, in any order, and in parallel. You can use a multipart upload for objects from 5 MB to 5 TB in size. For more information, see [Uploading Objects Using Multipart Upload API](https://docs.aws.amazon.com/AmazonS3/latest/dev/uploadobjusingmpu.html).

Background
==========
When uploading large file more than 5 GB, we have to use multipart upload by split the large file into several parts and upload each part, once all parts are uploaded, we have to complete the multipart upload.

What if I want to use presigned url?
At the time I documented this, AWS SDK presigned operation doesn't support to provide additional parameters to get presigned url for part upload. The two parameters need to perform part upload operations are partNumber and uploadId.

Problem
=======
How do I get presigned url for each part?

Solution
========
- Start a multipart opeartion using any AWS SDK and get UploadId; Python Boto3 [Create Multipart](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.create_multipart_upload)

- Create a Canonical Request for each part upload using Signature Version 4. For more information, see [Signing AWS Requests with Signature Version 4](https://docs.aws.amazon.com/general/latest/gr/sigv4_signing.html)
- Example of Part Upload Canonical Request

<pre>
PUT
 	/[Replace with S3 Object Key]
 	uploadId=[Replace with Upload Id]&partNumber=[Replace with 1,2,3...]
 	date: Fri,28 Sep 2018 11:53:05 UTC
 	host:[Your bucket name].s3.amazonaws.com
 	x-amz-content-sha256:a973958be9796e1828804c04894509fdf6b70d2c77b62b49bd2cef25674c032b
 	x-amz-date:20180928T115305Z

 	date;host;x-amz-content-sha256;x-amz-date
 	a973958be9796e1828804c04894509fdf6b70d2c77b62b49bd2cef25674c032b
</pre>

- Upload each part using client; and add ETag provided by each part upload. e.g Parts [{'PartNumber': 1, 'ETag': 'string'}, {'PartNumber': 2, 'ETag': 'string'}]

* Complete multipart operation using UploadId and Parts. Python Boto3 [Complete Multipart](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.complete_multipart_upload)

Discussion
==========
This will provide you an overview to understand and use of AWS documentation to implement a solution.

Have you implemented this solution?

Yes, I implemented using serverless architecture, so I created Lambda function which performs operations create_multipart(), upload_part() and complete_multipart() for the backend; attached Lambda function to API Gateway; Created client library to make API calls; Added multithreading to client library to upload multiple parts at the same time.

Is this suppose to be written in Python?

No, Choose your favorite language and [AWS SDK](https://aws.amazon.com/getting-started/tools-sdks/).
