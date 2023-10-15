What are the benefits of syncing WordPress media files to Amazon S3?

WordPress includes a built-in media library that offers useful features like image resizing, cropping, and more. While hosting images in the WordPress media library is convenient, as your website grows and the number of items increases, it may be more beneficial to move your image hosting to another location, such as AWS S3. This is mainly because:

    As the size of the wp-content/upload media files increases, backing up the site becomes more challenging and can cause issues when restoring it.
    It is inconvenient to download all the media library files to a local computer during development.
    WordPress can generate images in multiple sizes, but we cannot request an exact size. Creating multiple images for each size takes up a lot of unnecessary space.

Before discussing our tutorial, it is worth noting that Amazon has a tutorial available that demonstrates how to set up the sync using a plugin. However, it is important to keep in mind that the plugin approach may modify the way images are saved in WordPress. In contrast, our tutorial focuses on the development approach and provides step-by-step instructions for setting up the sync in a customized manner while preserving a copy of the images in the WordPress media library.

Step 1: Setting Up AWS Resources for Syncing WordPress Media Files to S3

We have a few steps to perform an aws to create the bucket on the user.

    Set up an S3 bucket where the image can be stored.
    Create an IAM user account.
    Establish a policy that grants access to the S3 bucket.
    Attach the policy to the user created in step 2.

What is cloudformation?

To carry out these steps, we will utilize AWS CloudFormation. CloudFormation is a method for provisioning AWS resources using code. Infrastructure as Code (IaC) offers a more convenient approach to creating AWS resources compared to using the User Interface (UI) or Command Line Interface (CLI) commands.

Utilizing CloudFormation is quite straightforward. Simply go to the AWS Console and navigate to the CloudFormation section. Choose “Create template in Designer” and click on the “Create template in Designer” button.

NB: Use the CF template in the file: create-s3-bucket-iam-user.json

Copy the given JSON template into the “Template window,” and a visual representation of the resource diagram based on the template will appear. Next, click on the “Create stack” button located in the upper right corner.

AWS will upload the template to an S3 link. Click on “Next” to instruct AWS to create the resources based on the template.

On the following screen, you’ll see a display similar to the one shown. In just a few minutes, the resources will be created

Once the creation process has finished, you can find a link to the AWS::IAM::User resource within the “Resources” tab.

Now, navigate to the IAM Edit User screen by clicking on the IAM resource. From there, create an access key for the user that CloudFormation has just created.

You can also create an access key using the AWS CLI command: `aws iam create-access-key — user-name wordpress-s3-user.`

Once we have access key and secret key, we must exercise caution in storing these credentials since they grant access to AWS resources. Therefore, we should ensure that we store them in a secure location.

Step 2: Configuring AWS Settings in WordPress and Activating the Plugin

Now that we have completed the setup of the AWS S3 bucket and user, we can proceed with configuring WordPress for synchronization. This process involves three steps:

    Setting the access key and secret key in the wp-config.php file
    Adding the code to sync uploaded or deleted media files to S3
    Installing the plugin and testing the sync.

To set the access key and secret key in the wp-config.php file, we simply need to add the following lines to the wp-config.php file located in the WordPress root directory, using the details obtained earlier in step 5:

define('S3_ACCESS_KEY', 's3-access-here');
define('S3_SECRET_KEY', 's3-secret-key-here');

The plugin below contains the code required for synchronization. You can copy this code into your theme or custom plugin. Note that this code is currently located in a plugin for demonstration purposes, but can be placed within any theme or plugin.

Before installing the plugin, we can confirm that there are currently no files in the S3 bucket by running the following command:

aws s3 ls s3://<BUCKET-NAME>

Once confirmed, we can install the plugin and upload a file via the media upload UI to test the synchronization.

The full plugin code can be find in the folder wp-aws-s3-sync-main

Before installing the plugin, update the following lines with your bucket information and the AWS region of the bucket:

$this->bucket_name='your-bucket-name'; 
$this->bucket_region = 'aws-region';

After activating the plugin, following a file upload the file will be removed from the WordPress server disk after the synchronization process is finished. If you experience any problems, enable WordPress debug mode by setting it to true and investigate the errors. The issue could be a result of incorrect access key configuration or an error originating from AWS.

At this stage, if you uncomment the line

//return $this->get_s3_url($relative_path);

and comment out

$this->get_aws_image_handler_cdn_url($relative_path);

you should see the images in the Media Library being sourced from the S3 bucket. After testing, revert the code to its original state.

Step 3: Setting up CDN to Direct to the S3 Bucket

By serving images through the AWS solution “AWS Serverless Image Handler” rather than directly from the S3 bucket, we can enjoy reduced latency and lower data transfer costs. This service provides a Content Delivery Network (CDN) for the bucket, which offers the added benefit of enabling dynamic size requests for each image. This approach results in improved flexibility and efficiency.

Follow these steps:

    Visit this page: https://docs.aws.amazon.com/solutions/latest/serverless-image-handler/template.html
    Click “View template” to download the template file to your computer.
    Upload the downloaded template to CloudFormation.
    Create a new stack using the uploaded template.
    After the stack is successfully created, go to the “Output” tab, where you’ll discover several variables generated from the created resources.

Copy the ApiEndpoint value from the CloudFormation output and paste it into your plugin code, setting it as the value of this line in the plugin file.

$this->cdn_url = 'cloudfront-url-from-cloudformation'; 

That’s it! Now, uploads on your website will be served from the S3 bucket via the CDN.
Requesting Various Image Sizes from the CDN

To explore all the available options for image requests, go to the `DemoUrl` value found in the CloudFormation output tab.

As demonstrated in the screenshot below, you can set different variables for the request and see a preview of the resulting image in the specified size. Modify the code of the JSON base64 encoded parameters to adjust the options for the image request according to the context in which you use the image.

An example of code to request a resized image URL using our class would resemble the following:

public function get_resized_image_url(string $relative_path, int $width, int $height): string {
    $json = json_encode([
        "bucket" => $this->bucket_name, 
        "key" => "uploads" . $relative_path,
        "edits" => [
            "resize" => [
                "width" => $width,
                "height" => $height,
                "fit" => "fill"
            ]
        ]
    ]);

    $json_base64 = base64_encode($json);
    $url = $this->cdn_url . "/" . $json_base64;
    return $url;
}


Final notes

This plugin and CloudFormation templates are for educational purposes of demonstrating a concept. Do not use them in production as is.