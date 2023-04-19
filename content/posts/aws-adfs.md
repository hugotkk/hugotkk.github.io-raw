---
title: "Setting up ADFS Login in AWS with Windows Server 2012"
date: 2023-04-19
tags:
- aws
- adfs
- windows
- ad
- identity-provider
- iam
---

* Create a new Windows Server 2012 instance and install the following Roles and Features: 
	- DNS
	- ADFS
	- AD

* Promote the server to a Domain Controller and create a new forest. I named mine `hhuge9.com`.

* Follow a tutorial (such as this one: https://www.youtube.com/watch?v=9eq3IeDAkvA) to configure ADFS.

* You can skip the process of generating the certificate in those tutorial, as it can be self-signed and needs to be in PFX format for ADFS to use it.

* If you only have an OpenSSH key and certificate, you can convert it to PFX format using the following command:
	
	`openssl pkcs12 -export -out certificate.pfx -inkey hhuge9.com.key -in hhuge9.com.crt -certpbe PBE-SHA1-3DES -keypbe PBE-SHA1-3DES -nomac`

* Then, copy the resulting PFX file to the Windows server, double-click it, and start the import process. Once it's imported, the certificate should be shown in the ADFS Wizard.

* In Active Directory, create a new user (I named mine `tsek`) and include their email address (which is a required field for the RoleSessionName).

* Create a group called `AWS-437735673474-ADFS-Admin` (replace `437735673474` with your actual AWS account ID and `ADFS-Admin` with the name of the IAM role you want to assume in AWS).

* Add the `tsek` user to the `AWS-437735673474-ADFS-Admin` group.

* Follow the tutorial at https://aws.amazon.com/blogs/security/aws-federated-authentication-with-active-directory-federation-services-ad-fs/ to set up a relay party trust on ADFS and add claim rules.

* In AWS IAM, add an identity provider and set the IAM role to "ADFS-Admin".

* Use the ADFS login page (https://hhuge9.com/adfs/ls/idpinitiatedsignon) to log in to AWS using your AD credentials.
