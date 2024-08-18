# Deploying simple static website via AWS services
The goal of this project is to deploy simple webpage on AWS using Infrastructure as Code (IaC) approach. Here, we'll go through deployment process of highly available, scalable and secured website.
Result website: [tipofyzik.online](https://tipofyzik.online/)


Architecture:
![AWS drawio](https://github.com/user-attachments/assets/5dcd04e3-4581-4a05-b267-8ffd91c3966c)

## 1. Prerequisites
1. The very first thing you need to host your website is custom domain name. There are a variety of companies that offer registration of custom domains. You can choose companies such as Google and Amazone, especially if you require a compatibility with these platforms. But for small projects like this you can buy domain name on [namecheap.com](https://www.namecheap.com/). It offers a variety of features for customizing your domain for pretty low fee.
![image](https://github.com/user-attachments/assets/c95cdcc0-6565-4c23-b9ce-505e458802e2)  
2. The second thing that should be taken into account is the registration on (Amazon Web Services)[https://aws.amazon.com/]. To use AWS resources you are required to register in their system.  
3. Download both folders from this github page **templates** and **webpage** and save it wherever you want on your pc.
4. To work with YAML files you need some code editor. I use [VS Code](https://code.visualstudio.com/) because it supports a lot of languages and has a vast variety of extensions that allows to work a variety not built-in tools.

## How to use
Once you acquired custom domain name and registered on AWS you can start setting up and managing AWS resources. The process of deployment is divided into 5 steps. Let's go through each of them:
**1. Setting up S3 buckets**  
Open the first file in templates folder, **1-S3Buckets.yaml**, and specify the parameters. Namely, you shoud specify you domain name and subdomain that you want to use.  
**Note!** In this implementation, the website is accessible via the domain name tipofyzik.online, and requests to the subdomain www.tipofyzik.online are redirected to the root domain (tipofyzik.online).
Once you specified your domains, open CloudFormation on AWS. This service helps to set up and manage AWS resources that we use. Click on "create stack" and upload the first template with already specified parameters. After this, give the name to your stack and leave all other settings untouched.  
![image](https://github.com/user-attachments/assets/507b83ee-7c89-42fb-ab7f-a32b9d62159e)  
Once the stack is created, open AWS S3 bucket that is responsible for the **root domain** (e.g., example.com). Upload all HTML pages from the webpage folder.  

**2. Setting up Route 53**  
In this step, we create public hosted zone and configuring our DNS. Open the second template, **2-Route53.yaml**, specify the root domain and upload this file on CloudFrontation. Once the public hosted zone is created click at the stack and open "outputs" tab. You can find **Route53HostedZoneNS** outputs that contains nameservers to connect our domain and Route 53. How to do this [read here](https://aws.plainenglish.io/how-to-connect-your-domain-from-namecheap-to-amazon-route-53-840bc745ce67#:~:text=How%20to%20connect%20my%20domain%20to%20Route%2053%3F).  

**3. Issuing SSL certificate**  
Here, we want to insecure the connection between the web browser and the web server. Open the **3-CertificateManager.yaml** and specify your donaim name and public hosted zone id here. You can find the latter parameter in outputs tab in the Route53 stack (where you took nameservers for you domain). Now, change the server form the current one to **US East (N. Virginia) us-east-1** on CloudFormation page (I recommend to open 2 pages with CloudFormations, since we'll require outputs from both pages later). **Don't skip this step! All certificates for the website must be issuied in US East region!**. Upload the file with specified parameter on CloudFormation in **US East region(!)**. Wait till certificate is issued, it usually takes from 10 to 30 minutes.  

**4. Creating CloudFront distribution**
Now, we creating a CloudFront distribution that is responsible for retrieving content from S3 bucket and caching it at Edge Locations. Open **4-CloudFront.yaml** and specify your domain, subdomain, and issued certificate ARN. You can copy this ARN from outputs of the third stack in the US East region. Then, upload this file on CloudFormation.  
**Note: 4th and 5th stacks you create in you original region!** 

**5. Creating records on Route53**
The last step is to create records to make our website available through the custom donaim. Once again, specify all the parameters (cloud distribution donaim name you can in the cloud front stack in **outputs tab**) and upload it. 

Congratulations! You've just deployed your first website!
![image](https://github.com/user-attachments/assets/6329c5c6-a8ce-4c43-8aab-c3d1f8d4d6b3)


## Sources
I highly recommend checking these resources that I relied on if something goes wrong or if you want to dive deeper into this topic. First two links lead to the website and youtube video that demonstrate how to deploy your website manually. The last link directs you to the project where the deployment process of a website via CloudFormation is explained. And can also find the source code there.  

[1] https://medium.com/@maksymyurchak/aws-s3-react-spa-cloudfront-route53-namecheap-how-to-host-your-website-with-domain-8e01c16187fc  
[2] https://www.youtube.com/watch?v=Bmuoqo_JY4g  
[3] https://dev.to/tiamatt/aws-project-module-4-use-cloudfront-distribution-to-serve-a-static-website-hosted-on-aws-s3-via-cloudformation-226m  
