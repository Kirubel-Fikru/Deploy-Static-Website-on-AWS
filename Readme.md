The cloud is perfect for hosting static websites that only include HTML, CSS, and JavaScript files that require no server-side processing. This project  has two major intentions to implement:

* Hosting a static website on S3 and
* Accessing the cached website pages using CloudFront content delivery network (CDN) service. Recall that CloudFront offers low latency and high transfer speeds during website rendering.

## Prerequisites:
* AWS Account

## Dependencies
### Cloud Services
* Amazon Web Services S3 - Resource hosting and deployments
* AWS CloudFront - CDN for SPA
### Performance Tracking and DevOps Tools:
* AWS CloudWatch - Performance and Health checks

## Create S3 Bucket
1. Navigate to the “AWS Management Console” page, type “S3” in the “Find Services” box and then select `S3`.
2. The Amazon S3 dashboard displays. Click “Create bucket”.
3. In the General configuration, enter a `Bucket name` and a region of your choice. Note: Bucket names must be globally unique.
4. In the Bucket settings for Block Public Access section, uncheck the `Block all public access`. It will enable the public access to the bucket objects via the S3 object URL.
5. Click `Next` and click `Create bucket`.
6. Once the bucket is created, click on the name of the bucket to open the bucket to the contents.

![Bucket Created](Screenshots/s3-1.PNG)

## Upload files to S3 Bucket
1. Once the bucket is open to its contents, click the `Upload` button.
2. Click the `Add files` and `Add folder` button, and upload the Website Folder  content from your local computer to the S3 bucket.
    * Click `Add files` to upload the index.html file, and click `Add folder` to upload the css, img, and vendor folders.
    * Do not select the whole folder. Instead, upload its content one-by-one.

### Need help with uploading the files to S3?
Sometimes the local machine's network setting or firewall might block, or the browser's Adblocker may prohibit the file upload, such as buysellads.svg file. In such a case, here are the workarounds:

> #### Workaround 1
Try using Chrome browser, and turn off the Adblocker, if not already. Here are the steps to turn off the Adblocker in Chrome:

> ### Workaround 2
Use CLI commands to upload the files and folders:

1. Verify the AWS CLI configuration. If not configured already, use:

```
aws configure list
aws configure
aws configure set aws_session_token "<TOKEN>" --profile default
```
2. Upload Code

``` 
# Create a PUBLIC bucket in the S3, and verify locally as 
aws s3api list-buckets 
# Download and unzip the udacity-starter-website.zip 
cd udacity-starter-website 
# Assuming the bucket name is my-bucket-202203081 and your PWD is the "udacity-starter-website" folder 
# Put a single file. 
aws s3api put-object --bucket my-bucket-202203081 --key index.html --body index.html 
# Copy over folders from local to S3 
aws s3 cp vendor/ s3://my-bucket-202203081/vendor/ --recursive 
aws s3 cp css/ s3://my-bucket-202203081/css/ --recursive 
aws s3 cp img/ s3://my-bucket-202203081/img/ --recursive 
```
![Successfully uploaded starter code in s3](Screenshots/s3-2.PNG)

## Secure Bucket via IAM
1. Click on the `Permissions` tab.
2. The “Bucket Policy” section shows it is empty. Click on the Edit button.
3. Enter the following bucket policy replacing *your-website* with the name of your bucket and click `Save`.
```
{
"Version":"2012-10-17",
"Statement":[
 {
   "Sid":"AddPerm",
   "Effect":"Allow",
   "Principal": "*",
   "Action":["s3:GetObject"],
   "Resource":["arn:aws:s3:::your-website/*"]
 }
]
}
```
You will see warnings about making your bucket public, but this step is required for static website hosting.

***Note** -  We could have made the bucket private and wouldn't have to specify any bucket access policy explicitly. In such a case, once we set up the CloudFront distribution, it will automatically update the current bucket access policy to access the bucket content. The CloudFront service will make this happen by using the Origin Access Identity user.*

![Successfully Edited Bucket Policy](Screenshots/s3-3.PNG)
## Configure S3 Bucket
1. Go to the Properties tab and then scroll down to edit the Static website hosting section.
2. Click on the “Edit” button to see the Edit static website hosting screen. Now, enable the Static website hosting, and provide the default home page and error page for your website.

* *Enabling the static website hosting requires you to make your bucket public? For your customers to access the content at the website endpoint, you must make all your content publicly readable.*

3. For both “Index document” and “Error document”, enter `index.html` and click `Save`. After successfully saving the settings, check the Static website hosting section again under the Properties tab. You must now be able to view the website endpoint. Copy the website endpoint for future use.

![Successfully Edited Static Website Hosting](Screenshots/s3-4.PNG)

## Distribute Website via CloudFront
1. Select “Services” from the top left corner and enter “cloud front” in the “Find a service by name or feature” text box and select “CloudFront”.
2. From the CloudFront dashboard, click “Create Distribution”.
3. For “Select a delivery method for your content”, click “Get Started”
4. Use the following details to create a distribution:

| Field| 	Value |
| ----------- | ----------- |
| Origin > Domain Name | Don't select the bucket from the dropdown list. Paste the Static website hosting endpoint of the form <bucket-name>.s3-website-region.amazonaws.com |
| Origin > Enable Origin Shield | No |
| Default cache behavior | Use default settings |
| Cache key and origin requests | Use default settings |

5. Leave the defaults for the rest of the options, and click “Create Distribution”. It may take up to 10 minutes for the CloudFront Distribution to get created.
* ***Note**: It may take up to 10 minutes for the CloudFront Distribution to be created.*

6. Once the status of your distribution changes from “In Progress” to “Deployed”, copy the endpoint URL for your CloudFront distribution found in the “Domain Name” column.

* ***Note** - Remember, as soon as your CloudFront distribution is Deployed, it attaches to S3 and starts caching the S3 pages. CloudFront may take 10-30 minutes (or more) to cache the S3 page. Once the caching is complete, the CloudFront domain name URL will stop redirecting to the S3 object URL.*

![Successfully Distributed website using cloud front](Screenshots/s3-5.PNG)

## Access Website in Web Browser
***Note** - In the steps below, the exact domain name and the S3 URLs will be different in your case.*

1. Open a web browser like Google Chrome, and paste the copied CloudFront domain name (such as, dgf7z6g067r6d.cloudfront.net) without appending /index.html at the end. The CloudFront domain name should show you the content of the default home-page.
2. Access the website via website-endpoint, such as 

`http://<bucket-name>.s3-website.us-east-2.amazonaws.com/.`

3. Access the bucket object via its S3 object URL, such as, 

`https://<bucket-name>.s3.amazonaws.com/index.html.`

All three links: CloudFront domain name, S3 object URL, and website-endpoint will show you the same index.html content.

```
If we were not "hosting" the website on S3, we could have made the bucket private and host the content only through the CloudFront domain name. In such a case, we cannot access the private content using S3 object URL and website-endpoint.
```

![Website](Screenshots/s3-6.PNG)