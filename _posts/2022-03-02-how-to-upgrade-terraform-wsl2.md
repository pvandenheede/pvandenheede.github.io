---
layout: post
title: How to upgrade Terraform in WSL2
summary: The script to upgrade Terraform on WSL2
featured-img: 20220303-terraformversion
categories: [Terraform WSL2]
comments: true
mathjax: true
---

# How do I upgrade Terraform in WSL2?

Ever wanted to upgrade Terraform to a newer version, but forgot how to do it?
Here's how!

First, find the download link on the [Terraform downloads page](https://www.terraform.io/downloads). <br />
Pick the one for linux AMD64!

At the time of writing, this was the link I needed: https://releases.hashicorp.com/terraform/1.1.6/terraform_1.1.6_linux_amd64.zip

Now execute the following script and replace the link with the version you need!<br />
Make sure to execute the script into your home or a custom directory!

```
wget https://releases.hashicorp.com/terraform/1.1.6/terraform_1.1.6_linux_amd64.zip -O terraform.zip; 
unzip terraform.zip; 
sudo mv terraform /usr/local/bin; 
rm terraform.zip;
```

Explanation:
- **Step 1**: download the latest version of the Terraform executable package and save it as terraform.zip
- **Step 2**: unzip the file.
- **Step 3**: move it to your /usr/local/bin folder, this needs root privileges
- **Step 4**: remove the downloaded terraform.zip file again.
