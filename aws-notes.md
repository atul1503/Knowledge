# Services

1. Cloudfront is used as a CDN which caches your content in multiple edge locations and delivers it when its requested. But it must have some origin. The origin can be an s3 bucket or even a ELB. For s3, provide s3 bucket name to be used as origin, for elb, you can take help of cloud front so that elb doesnot have to server static content everytime.
2. 
