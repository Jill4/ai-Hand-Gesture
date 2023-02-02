# ai-Hand-Gesture
## Problem Overview

Many sought to believe a single camera wouldn't be as functional as depth aware cameras. However, using a simple 2D camera on an evalation board for the Renesas RZ/V2L (seen below), we dive into the functionality of a 2D camera by making it detect hand gestures.

## Hardware Requirements
  - Renesas [RZ/V2L Evaluation Board Kit]
  - Router that allows port-forwarding (if staying remote)
 
 ## Software Requirements
  - Edge Impulse account
  - A terminal with `ssh`
  - Yocto build for the board (instructions fromm Renesas and Edge Impulse)
  
  ## Familiarizing
  
  In order to get comfortable with the board and object detection, I watched a couple 
     videos of the camera being used. 
     (https://docs.edgeimpulse.com/docs/edge-impulse-studio/learning-blocks/object-detection/fomo-object-detection-for-constrained-devices) 
     If you want to try something smaller and simpler you can visit,    (https://docs.edgeimpulse.com/docs/tutorials/image-classification)
     
     
     
  ## 1. Set up and connect the Renesas RZ/V2L board ot the Edge Impulse Project
   
The [setup instructions](https://docs.edgeimpulse.com/docs/development-platforms/officially-supported-cpu-gpu-targets/renesas-rz-v2l)

 After the board was prepared, I forwarded it from port 2461 through my router
 and connected to it with `ssh` using the correct port and the user that I created:

 ```sh
 ssh -p2461 myuser@68.81.34.208
 ```


 
 ## 2. Connect the Renesas RZ/V2L board to the Edge Impulse Project
 
 To connect to the device, you may need to enter a password for you user depending
 on if you have one or not. If you are using the `root` user like they do in the
 Edge Impulse guide for the RZ/V2L board, then you won't need to bother with
 `sudo`ing the commands below.

 First I needed to set up `npm` to run as the `root` user and to install
 the [`edge-inpulse-linux`](https://github.com/edgeimpulse/edge-impulse-linux-cli) package.

 ```sh
 sudo npm config set user root
 sudo npm install edge-impulse-linux -g --unsafe-perm
 ```

 Now that everything is setup and ready to go, we run the Edge Impulse runner.

 ```ssh
 sudo edge-impulse-linux-runner
 ```

 You'll have to sign into your EI account to access your models.

## 3. Acquiring data and training
    
    You can start by collecting data right through your phone! When scanning the QR provided, it will take you to three options. From there you will press collect images and after labeling the data, it is now ready be trained!
    
    If the phone option doesn't work for you, it can also be done through a computer, any device with a data forwarder, or you can input data if you already have any. 
    
## 4. Deploy
All that is left is to test the model and load it onto the board.

 ```sh
 sudo edge-impulse-linux-runner --download downloaded-model.eim
 ```
 
       
 
 
 
 

   
  
 


