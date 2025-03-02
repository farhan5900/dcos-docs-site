---
layout: layout.pug
navigationTitle: Scaling a Service
title: Scaling a Service
menuWeight: 3
excerpt: Scaling a service using the UI and the CLI
enterprise: false
model: /1.14/data.yml
---

This tutorial shows how to scale a service using the UI and the CLI.

# Scale Your Service from the UI

1. From the **Services** tab, click the menu dots to the right side of your service to access **More Actions**. 

    ![Menu](/1.14/img/GUI-Services-More-Actions_Callout.png)
   
    Figure 1. More Actions menu

2. From the drop-down menu, select **Scale**.

   ![more actions menu](/1.14/img/GUI-Services-More-Actions-Menu.png)

   Figure 2. More Actions menu

3. In the **Scale Service** box, enter the total number of instances you would like, then click **Scale Service**.

   ![scale](/1.14/img/GUI-Services-Scale-Service-Instances.png)

   Figure 3. Choose number of instances

4. From the Services tab, you can see your service scaling.

    ![service deploying](/1.14/img/GUI-Services-Scale-Confirmation.png)
    
    Figure 4. Service scaling

5. Click the name of your service to see your scaled service.
   ![post scale](/1.14/img/GUI-Services-Scaled-Service.png)

   Figure 5. Services list 

# Scale Your Service from the CLI

You will need your `app-id` for this procedure.  

1.  Enter the following command from the CLI, replacing `app-id` with your own value and  `total_desired_instances` with your own number:

    ```bash
    dcos marathon app update <app-id> instances=<total_desired_instances>
    ```

1.  Enter this command to see your scaled service.

    ```bash
    dcos task <task-id>
    ```


For example, this task is scaled to 6 instances:

```bash
dcos marathon app update basic-0 instances=6
dcos task basic-0
NAME     HOST        USER  STATE  ID                                            
basic-0  10.0.0.10   root    R    basic-0.1c73e448-0b47-11e7-a8b6-de4438bbb8f0  
basic-0  10.0.1.101  root    R    basic-0.1c739626-0b47-11e7-a8b6-de4438bbb8f0  
basic-0  10.0.1.41   root    R    basic-0.1c736f14-0b47-11e7-a8b6-de4438bbb8f0  
basic-0  10.0.1.92   root    R    basic-0.12d5bbc2-0b47-11e7-a8b6-de4438bbb8f0  
basic-0  10.0.1.92   root    R    basic-0.1c73bd37-0b47-11e7-a8b6-de4438bbb8f0  
basic-0  10.0.3.180  root    R    basic-0.1c739625-0b47-11e7-a8b6-de4438bbb8f0
```
