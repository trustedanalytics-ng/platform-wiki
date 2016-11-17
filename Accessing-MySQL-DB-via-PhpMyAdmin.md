---
title: MySQL Credentials
keywords: MySQL Credentials
last_updated: 'November, 2016'
tags:
  - MySQL Credentials
summary: How to get your MySQL credentials for PhpMyAdmin 
sidebar: mydoc_sidebar
permalink: mysql_credentials.html
folder: mydoc
published: true
---

## Getting your MySQL Credentials for PhpMyAdmin

Proceed as follows:

1. Clone the PhpMyAdmin tool:

      `git clone https://github.com/cloudfoundry-community/phpmyadmin-cf`

9. Log onto a platform with your cf client. (Choose an appropriate organization.)

9. Go to the phpmyadmin folder and push the downloaded application to the target organization. 

9. Log onto the TAP platform using your TAP user/password credentials and choose the target organization.

9. Go to **Marketplace>Services** and create a new instance of MySQL database. 

9. Go to **Applications** and locate your instance of PhpMyAdmin app.

9. Bind your instance of MySQL DB to your instance of PhpMyAdmin. 

9. Return to the cf command line and run the following command: 

    `cf env <the name of your instance of PhpMyAdmin>`

 ![Getting_Credentials_for_MySQL](/images/MySQL_Getting_Credentials_MyPhpAdmin_v7.png)

9. Get the credentials for your MySQL DB.

9. Return to your browser and follow the URL that is exposed by your instance of PhpMyAdmin tool.

9. You should be presented with the PhpMyAdmin login page. Once you enter credentials to your instance of MySQL DB, you will have access to your MySQL DB instance. 


