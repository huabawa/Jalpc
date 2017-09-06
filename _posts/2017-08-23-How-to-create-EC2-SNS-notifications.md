## How to set up EC2 SNS notifications
Follow the steps below to set up EC2 start,stop,terminate SNS notifications.

1. Go to SNS in AWS Services.
2. Go to Topics. Create new topic.
3. Choose a topic name, like EC2-Start-Stop
4. Select (tick) the topic. Under Actions, select Subscribe to topic.
5. Select a protocol. Email is preferred. For endpoint, type in your email.
You will a receive an email asking you to confirm the subscription.
6. Go to CloudWatch in AWS Services.
7. Click Events and then click Create rule.
8. Under Service Name, choose EC2.
9. Under Event Type, choose EC2 Instance State-change Notification
10. Choose any state or specific states depending on what you want.
11. Click Add Target. Scroll down and select SNS topic.
12. Selec the topic that you had just created.
13. Click Configure Details. Write a name and description. Then, click Create rule.

