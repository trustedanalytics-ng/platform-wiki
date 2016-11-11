---
title: Platform Administration
keywords: platform administration
last_updated: 'October, 2016'
tags:
  - Platform Administration
summary: How to create new orgs, invite users, delete orgs and users. 
sidebar: mydoc_sidebar
permalink: administration_manageorgsusers.html
folder: mydoc
published: true
---

This page shows you how to create a new organization, how to invite users to an existing organization, and how to delete organizations (and users from organizations).

# Adding a new organization

> You must be a system admin to do this.

Here are the steps to create a new organization:

1. Navigate to **User management** and click on **Manage organizations**.

2. Choose the **Add organization** tab.

3. Specify a name for the organization in the **Name** field.

4. Click the **Add organization** button.
![](/images/Add_Organization_v7_Crpd.png)

5. TAP creates the organization and displays a confirmation message in the upper right of the screen.

For more information about organizations and spaces, go [here](https://docs.cloudfoundry.org/concepts/roles.html) (Cloud Foundry page).

# Deleting an organization

> You must be a system admin to do this.

Here are the steps to delete an existing organization:

1. Navigate to **User management** and click on **Manage organizations**.

2. Choose the **Organizations List** tab.

3. Select the organization you wish to delete on the left side.

4. Click the red **Delete organization** button on the lower right side.
![](/images/Delete_Organization_v7_Crpd.png)

5. Confirm the deletion.

6. TAP deletes the organization and displays a confirmation message in the upper right of the screen. Refresh your browser after several seconds to see the deletion reflected on the screen.

#Adding users to an organization

> System admins can add users to any organization. Others can add users to organizations they manage. If the user does *not* already have a user account on the platform, TAP generates an invitation email to the user.

Here are the steps to invite a user to an existing organization: 

1. Navigate to **User management** and click on **Manage organization users**.

2. Choose an organization from the dropdown menu at the upper right of the screen.
![](/images/Organization_Selection_v7_Crpd.png)

3. Choose the **Add user** tab.

4. Enter the email address of the user you wish to add.

5. The only role available is the TBD role.

6. Click the **Add user** button.
![](/images/Add_User_v7_Crpd.png)

For existing platform users, TAP will notify you that the user has been added to the organization. But *you must notify the user* of this addition, as TAP does *not* send any notifications.

If the user does *not* yet have a TAP account, TAP sends an email invitation to the user. For the user view of this process, read Responding to an Initial Organization/Space Invite [here](accessing_account.md).

For more information about organizations, [go here](https://docs.cloudfoundry.org/concepts/roles.html) (Cloud Foundry page).

#Deleting users from an organization

> System admins can delete users from any organization. Others can delete users from organizations they manage.

Here are the steps to delete a user from an existing organization:

1. Navigate to **Manage organization users** (or **Manage space users**).

2. Choose the organization (and space) to manage from the dropdown menu in the upper right of the screen, as shown below:
![](/images/Space_Selection_v7_Cropped.PNG)

3. Click the **x** (delete symbol) next to user you wish to delete.

4. Confirm the deletion.
![](/images/Delete_User_Organization_v7_Cropped.png)

5. TAP deletes the organization (or space) and displays a confirmation message in the upper right of the screen. Refresh your browser after several seconds to see the deletion reflected on the screen.

>This only deletes the user from the organization; their user account still exists at the platform level.
