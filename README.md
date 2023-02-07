# ai-Hand-Gesture
## Problem Overview

Many sought to believe a single camera wouldn't be as functional as depth aware cameras. However, using a simple 2D camera on an evalation board for the Renesas RZ/V2L (seen below), we dive into the functionality of a 2D camera by making it detect [hand gestures]().


## Hardware Requirements
  - Renesas [RZ/V2L Evaluation Board Kit]
  - Router that allows port-forwarding (if staying remote)
 
 ## Software Requirements
  - Edge Impulse account
  - A terminal with `ssh`
  - Yocto build for the board (instructions fromm Renesas and Edge Impulse)
  
  ## Familiarizing

  
  In order to get comfortable with the board and object detection, I reviewed this [document](https://docs.edgeimpulse.com/docs/tutorials/object-detection) to get an idea of how the camera works. If you want to try something smaller you can follow any of these [tutorials](https://docs.edgeimpulse.com/docs/tutorials/image-classification) ideos of the   being used.  
     
     
     
  ## 1. Set up and connect the Renesas RZ/V2L board ot the Edge Impulse Project
   
The setup instructions I used went smoothly and thoroughly going through the instrctions will get you where you need to do. 

The process is:
- Either be running [Ubuntu](https://ubuntu.com) 20.04 or have it running in a `docker` container.
- Download the [RZ/V Verified Linux Package [5.10-CIP]](https://www.renesas.com/eu/en/software-tool/rzv-verified-linux-package), the [RZ/V2L DRP-AI Support Package [V7.20]](https://www.renesas.com/eu/en/products/microcontrollers-microprocessors/rz-arm-based-high-end-32-64-bit-mpus/rzv2l-drp-ai-support-package), the [RZ MPU Graphics Librarry Evaluation Version for RZ/V2L](https://www.renesas.com/eu/en/products/microcontrollers-microprocessors/rz-arm-based-high-end-32-64-bit-mpus/rz-mpu-graphics-library-evaluation-version-rzv2l) (optional but recommended), and the [RZ MPU Video Codec Library Evaluation Version for RZ/V2L](https://www.renesas.com/eu/en/products/microcontrollers-microprocessors/rz-arm-based-high-end-32-64-bit-mpus/rz-mpu-video-codec-library-evaluation-version-rzv2l) (also optional, but strongly advised for this project).
- create a folder for all of this and decompress everything there; e.g.,
```
mkdir ~/rzv_vlp_v3.0.0
cd ~/rzv_vlp_v3.0.0
unzip ~/RTK0EF0045Z0024AZJ-v3.0.0-update2.zip
tar zxvf ./RTK0EF0045Z0024AZJ-v3.0.0-update2/rzv_bsp_v3.0.0.tar.gz
unzip ~/RTK0EF0045Z13001ZJ-v1.xx_EN.zip
tar zxvf ./RTK0EF0045Z13001ZJ-v1.xx_EN/meta-rz-features.tar.gz
...
```
- patch the VLP from your working directory with
```
patch -p1 <./RTK0EF004Z0024AZJ-v3.0.0-update2/rzv_v300-to-v300update2.patch
```
- Edit `poky/meta/conf/sanity.conf` to allow `root` user to run bitbake from your `docker` container by commenting out the sanity check.
```
# INHERIT +=  "sanity"
```
- Modify the BSP layer so the build will work
```
mv -n ../meta-renesas/include/core-image-bsp.inc ../meta-renesas/include/core-image-bsp.inc_org # move the original
grep -v "lttng" ../meta-renesas/include/core-image-bsp.inc_org >> ../meta-renesas/include/core-image-bsp.inc # only include the "lttng" lines in the new file
```
- Start a build enviroment with
```
source poky/oe - init-build-env
```
- Copy the Yocto config template
```
cd ./build
cp ../meta-renesas/docs/template/conf/smarc-rzv2l/*.conf ./conf
```
Edit `local.conf` to include at least the tools necessary for the Edge Impulse CLI runner, but also any other packages you might want to install, or any options you might need.
```
# Select CIP Core packages
CIP_CORE = "0"

# Add packages
IMAGE_INSTALL_append = " \
    nodejs \
    nodejs-npm \
    "

# Add recipes
BBMASK = "meta-renesas/recipes-common/recipes-debian"
```
- Assuming you are still in the `build` directory, kick off the build!
```
bitbake core-image-weston
```
Assuming everything went to plan, you now have a Yocto image in `build/tmp/deploy` that you can flash to an SD card. Take a look at [section 4 here for explicit instructions](https://renesas.info/wiki/RZ-V/RZ-V2L_SMARC). If you are using docker, you'll need to copy the `build` directory to the host
```
docker cp <containerId>:/file/path/within/container /host/path/target
```
Put the SD card into the read on the carrier board (not the SMARC board) and we are ready to go!

 ## 2. Connect the Renesas RZ/V2L board to the Edge Impulse Project
 After preparing the board I forwaded from port 2461 through my router and connected it to with `ssh` and the user that I created:
 ```
 ssh -p2461 my user@68.81.34.208
 ```
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

## 3. Acquiring data and design an impulse 
    
You can start by collecting data right through your phone, through image classification. After scanning the QR provided, it will take you to three options. From there you will press collect images and after labeling the data, it is now ready be trained!

If the phone option doesn't work for you, it can also be done through a computer, any device with a data forwarder, or you can input data if you already have any. 

Here are the Hand Gestures to try:
- Horns
- Index finger
- Two finngers
- superb

After collecting the data, be sure to look if your train/Test Split is balanced. In this case it has a 82%/18% split. 

## Training 
At this point, we have our images ready to use to train our machine learning model.

First off, click Object detection. Change the 'Target' in the top left to be 'Renesas RZ/V2L (with DRP-AI accelerator)'. Since I am using the FOMO model, I chose 'FOMO (Faster Objects, More Objects) MobileNetV2 0.35'. 

 Then click Start training. Once the model is done, you can see the accuracy numbers like so:

## Design an impulse
Now it's time to design our impulse! First off, click Create impulse in the left navigation. Using FOMO, we can set our 'Image width' and 'Image height' to anything as long as it is square. For example, I used a 48x48 resolution.

 If you have been using FOMO as I have been for this tutorial, choose any square resolution.

Set the 'Resize mode' to 'Fit shortest axis'. Click Add a processing block, and click the Add button to the right of 'Image'. Click Add a learning block, and click the Add button to the right of 'Object Detection (Images)' with the author 'Edge Impulse'. Then click Save impulse.

    
## 5. Deploy
All that is left is to test the model and load it onto the board. 

 ```sh
 sudo edge-impulse-linux-runner --download downloaded-model.eim
 ```
 
       
 
 
 
 

   
  
 


