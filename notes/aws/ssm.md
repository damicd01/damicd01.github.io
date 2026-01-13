**SSM Oveview**

- It is a service that is designed to help manage EC2 and on-prem instances
- It allows for patch automation as well as operational insight for running instances
- Works for both Linux and Windows
- It integrates with other AWS services such as:
  - Config 
  - Cloudwatch
- It is a free service
- How it works:
  - First you need to instal an SSM agent onto your instances which will allow for communication back to the SSM service (this comes pre-built in AWS Linux AMI)
  - Once the SSM agent has been installed it will automatically be configured to talk back to the service.  If not then there is a comms issue or maybe the instance doesnt have the correct IAM permissions or the SSM agent is not running
