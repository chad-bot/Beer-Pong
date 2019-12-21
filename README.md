
# Introduction
Our goal for this project was to allow a Baxter robot to play beer pong.
For a human with decent hand-eye coordination, this game is simple enough to play while intoxicated. However, for a Baxter robot, it's much more complicated. 

Our project is mostly interesting in the way our sensing/vision component translates to actuation. Without using multiple AR-tags, we have to detect a the position and orientation of 6 red cups on a table in order to properly aim the robot. Other intriguing parts of our project are the modeling and the hardware. We model a ball's projectile motion based on its angle and initial velocity, which was calculated using data from test shots of our gun. 
    
We can break down the game of beer pong into three main problems: 

1. **Computer vision**, to recognize and locate red Solo cups on a table.
2. **Path planning**, to aim Baxter's right arm according to the best trajectory.
3. **Custom end-effector hardware**, to let Baxter "throw" a ping-pong ball with consistency and accuracy.


#### Real World Application
Besides helping you win at parties, CHAD has several useful applications in the robotics industry.
Our cup detection could be used in places such as laboratories or warehouses, where precise identification of containers and their orientation can help automate the process of moving objects from one place to another. For example, Amazon is greatly automating their fulfillment centers and relying on robots to do simple moving tasks. 
Our trajectory calculation of the ball makes use of a concept that can be applied to any weapons technology that involves projectiles. For example, a robot can determine if its current position is the best position to take a shot at a certain target, or if it should move itself to another location that allows for a better trajectory.


# Design

## Criteria
Before any implementation, we decided that we would judge the success of CHAD on these basic criteria:

- Precisely shooting the ping pong ball each turn
- Consistent recognition of cups and calculation of trajectory
- Robust end-effector hardware.

The desired functionality for CHAD is to shoot at different configurations and distances of cup targets. At a medium range, and with 6 cups in a diamond formation, we hoped to achieve a 50% cup hit rate.  We believe this percentage represents our team members’ average sober cup pong ability.

## In depth design (please include pics!!)
### Vision
For the vision component, we used an Intel RealSense camera and image processing to identify cups within the frame. Then for each cup we compute a center coordinate, corresponding to the center point of the circle created by the rim of each cup. We then return the coordinates of this point to the targeting component. 
    
Hardware: Mina todo
    
Planning: mina/ayrtun todo
    
### Trajectory calculation: 
Upon receiving the coordinates of the cup, we use projectile motion equationsto find the appropriate launch angles.\
First determine the Yaw Angle (ψ)

<img src="images/yaw.jpg" width="400">

Pitch Angle (θ)\
Given the initial velocity of the ball v_i, distance to target ∆d, and height to target ∆z, we can find the pitch angle θ.

<img src="images/pitch.jpg" width="400">




## Design choices and trade-offs
Our main trade-off was our end-effector, which was used to achieve the throwing action of the ball by the robot. Using a plastic gun helped us achieve initialization of the force acting on the ball. Otherwise, it was very difficult to give initial velocity to the ball by mimicking human throwing action based on the torque that is achieved by the arm movements. However, the gun was not very stable and consistency of the velocity generated by the gun is questionable.
	
For the vision, we decided to use a Intel RealSense camera. In order to focus on the target, we cropped the image that is received from the camera. This is because we wanted to reduce noise. This design choice helped us focus on the target.        
    
For the targeting, the design choice we made is to set an initial position of the robot arms. We wanted to aim at the target both using the pitch angle as well as yaw angle. For the yaw angle we aligned the end-effector with the robot's shoulder so that by changing the angle at shoulder joint, we were able to control the yaw angle.

### Effects of our design choices
- Robustness: The vision component works efficiently for a fixed configuration, as it has parameters (e.g. cropping) which are specific to the cup configuration. However, once that is set, from our experiments, we have seen that it identifies cups and the targetting points effectively and efficiently.
- Durability: Using a plastic toy gun seemed unreliable at first, so we needed to conduct extensive test shots with the gun in order to determine how precise it is, and also calculate an initial velocity of the ping pong ball.
- Efficiency: Our design is both time and cost efficient. Our decision of using a publisher-subscriber model ensures that CHAD is always listening for target coordinates, which are transferred from the cup detection node to the baxter gripper and arm actuation nodes. CHAD's path planning improves efficieny by only ever moving two joints, the shoulder and the wrist, to aim the arm to a selected cup. The only purchases required were a set of plastic ping pong guns and ping pong balls ($10 on amazon) and red solo cups, which we already had. Other materials (zip ties, rubber bands, tape, etc.) were negligible.


# Implementation

## Hardware & Parts
We ordered a plastic gun from Amazon to be our ping pong ball actuator. We used zip ties and rubber bands to properly secure the gun. We set up red solo cups on a black tablecloth for the targets.

todo: insert gun pic
## Software
### Vision Component
For the vision component, we cropped the image generated by the RealSense camera. We determined the range to crop by arbitrarily setting width and height bounds and tilting our camera such that the cups do not get cropped out. Thereafter, we used a mask to filter out only the colors red and white (as these are the colors of the cup). Next, we used erosion and dilation to remove any noise and to help make the detected rims more circular in shape. Following, we founds the contours of the image by using Canny edge detection, then calling OpenCV's contour function. This provided us with the figures of all the cups. From there, we take only the circular contours, as these will be the cup rims, and for each circular contour we calculate its moment and draw it on the image as a circle. Since the targeting component simply needed one point, we generate just one moment and return a binary image containing only said moment. Then, we use lab6 pointcloud projection code to create a pointcloud, which is just one point (the moment) and we attach a TF Frame centered at said point. *CHECK* An example of this processing can be seen below, for an image with 1 cup.
<img src="images/VisionExample.png" width = "400">

### Pipeline (system diagram)

### Description of steps - shooting the ball once


# Results
Our project worked well. In our proposal, we aimed for C.H.A.D.Bot's accuracy to be greater than 50%, through all of our tests we averaged about 65% accuracy. Moreover, we were unable to reach greater than 75% accuracy due to the movement of the end-effector during the game. The main action we expected to perform was to able to show the target on the Baxter whenever we started the code and also make a shot which goes inside the red solo cup. We have achieved these two tasks during our demo.

## Video
<img src="images/thumbnail.jpg" width="400">
[Video Link](https://youtu.be/NxHdCN6QJ0c)

# Conclusion
## Discussion of Results
On average our accuracy meet our base goal of 50%. Unfortunately, we were never able to have it shoot all 6 cups continuously due to various errors, e.g. MoveIt failing, tracking of AR tag being lost midway, etc. However, we have videos, linked in the Additional Materials section, of it being able to target and shoot most of the 6 cups, individually. Thus, we have confidence that outside of said errors, C.H.A.D.Bot could maintain our desired accuracy over all 6 cups, continuously. 

## Difficulties
***TBD***

## Flaws & Hacks

One hack in our solution is the cropping of the frames within the vision component. While this simplified our image processing, since we only had at most 6 cups, it is not immediately generalizable to games with a large number of cups. Currently, to accommodate a greater number of cups we would have to change the cropping dimension, but a clearer, less-hacky solution is definitely possible. Specifically, we could experiment with the use of more noise reduction methods such as erosion and dilation, e.g. using larger kernels than that of which we currently use, image blurring, etc. Also we use a specific mask on the images, one that only saves pixels which are red or white, as we had a fixed color of the cups. We could experiment with avoiding this step and directly detecting the cups based on the contours and their relative area in the picture. This would allow the system to generalize to different types and colors of cups.
    
Moreover, we had a goal of C.H.A.D.Bot engaging in gameplay against an adversary. While we can still accomplish this through manual execution of the launch files, we were imagiging having a GUI for such gameplay and using our drunk noise function which would add noise to the vision component for each point scored by the opponent. Unfortunately, due to our struggles testing C.H.A.D.Bot when he was sober, we were unable to test such additional functionality. Ideally, we would test the drunk noise function, implement a GUI to fix this weakness. 
    
*INSERT MINA"S INGENUITY*

# Team
## Meet the Team
* Mina Beshay: Mina is a mechanical engineer and has machine shop experience as well as a strong design background.
* George Wang: George has some experience in ROS and industry experience in python.
* Akash Gokul: Akash's background is in computer science and machine learning. He does research in computer vision.
* Artun Dalyan: Artun has a background in signal processing as well as feedback systems.

## Contributions
* Mina Beshay: TBD
* George Wang: George wrote code for the gripper calibration and CHAD's trigger pull. He created the diagrams/visuals for this website.
* Akash Gokul: Akash was in charge of the vision component. He wrote most of the code for the vision component of the project. 
* Artun Dalyan: Artun helped Akash with vision for detecting target. Also, he worked on the targeting component and derived equations of physics which were coded for the C.H.A.D.Bot.

# Additional Materials
## Additional Videos:
*TBD*

## Code
 Code for our project can be found [here](https://github.com/chad-bot/CHADBot) (github.com/chad-bot/CHADBot).



