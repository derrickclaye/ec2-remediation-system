### System Overview

Undetected server failures can have catastrophic repercussions when gone undetected. In order to ensure that doesn’t happen, I leveraged ServiceNow in order to configured a remediation system that detects when a server has failed and deals with it accordingly. When a server’s status changes from ‘ON’ to ‘OFF’ this triggers the remediation workflow.  The workflow is comprised of 3 steps. First I leveraged AI Search to return articles that outline how to solve a failed EC2 instance. That article is then sent to the appropriate personnel via Slack. Lastly an incident record is created to document the failure. That completes the workflow, and now after referencing the article(s) the dev-ops engineers now know what scripts to run in order to bring the failed server back online. 


<img width="1181" height="1159" alt="Diagram" src="https://github.com/user-attachments/assets/6aa490b9-178a-4159-a245-b01e29eb54cb" />




### Implementation Steps

I. In order to isolate this custom functionality the first step was to create a scoped application in ServiceNow. Some benefits of creating a scoped application include isolation and conflict prevention, improved manageability, and enhanced security.

II. The system is designed to facilitate a channel of communication between ServiceNow and AWS. When the remediation script is triggered from ServiceNow, it sends a request to an endpoint on the backend that is configured to fix the EC2 instance. The request has to be sent with the appropriate credentials which is why on the ServiceNow end I had to set up the necessary connection and credentials infrastructure.

III. Tracking the EC2 instances and remediation attempts on ServiceNow required the creation of two tables:

	- The EC2 instances table tracks the health of each instance. When the remediation script is triggered and the server is fixed,  the status of the particular EC2 is updated back to a healthy state.

	- The Remediation Logs tables tracks and logs remediation attempts. This table is also automatically populated when the remediation script is triggered. Some key fields on this table are Response Payload, HTTP Status and Success.

IV. The remediation workflow mentioned above in the System Overview required certain components:


<img width="582" height="328" alt="Screenshot 2025-09-12 at 9 35 26 AM" src="https://github.com/user-attachments/assets/ae01441f-0506-463e-8e21-72f0700f0a78" />


	- A knowledge article that outlines the steps needed to remediate the failed server. This article was placed inside an AI Search Application that was used in one of the workflow’s actions. 

	- The retrieved article data was then passed into a Slack action that sent the data to appropriate personnel via a web hook. 

	- The last step was to take relevant data from the EC2 record and create an Incident to document the failure. 

 V. At this point the Dev-Ops engineer knows that they must navigate to the EC2 record and press the UI Action EC2 Remediation Trigger.

 <img width="1465" height="301" alt="Screenshot 2025-09-12 at 9 49 20 AM" src="https://github.com/user-attachments/assets/53b14758-f377-4425-ad56-b5c08c0b1d77" />

 
This UI Action includes a script that leverages the GlideAjax API to call a Script Include from the client. The Script Include that was created is responsibe for making the API call that fixes the EC2 then creates a record in the Remediation Log table to detail the remediation attempt.

 
<img width="1334" height="420" alt="Screenshot 2025-09-12 at 9 47 50 AM" src="https://github.com/user-attachments/assets/cfb524c3-51c2-4932-9ade-c2e8b816bd01" />

 

### Optimization

In terms of basic infrastructure, I ensured certain table fields were made mandatory, created an Application Menu to contain the EC2 Instance and Remediation Log tables as well as ensured the Incident report created in the flow was as detailed as possible. 

<img width="314" height="274" alt="Screenshot 2025-09-12 at 9 36 39 AM" src="https://github.com/user-attachments/assets/e790a9cd-676b-42a0-894b-441ad5c84f11" />


### DevOps Usage

As a DevOps engineer you must have a dedicated slack channel in order to receive instructions on how to deal with failed EC2 instances. As you will see, in the event of a failed EC2 instance you will be sent an article that outlines the remediation steps which are:
	
	1. Navigate to EC2 instance table
	2. Press ‘Trigger EC2 Remediation’ UI action
	3. Check the Remediation Log to ensure you received a 201 HTTP Response Status Code

<img width="835" height="141" alt="Screenshot 2025-09-12 at 9 39 12 AM" src="https://github.com/user-attachments/assets/23b48846-da28-4864-96a3-ac97cec94bc7" />

 


