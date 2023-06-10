# Multivariant time series forcasting for teleoperation
## Required Hardware
- At least 2 HTC Vive Base Stations
- A VR headset compatible with open VR 
- HaptX gloves 
- A computer meeting the minimum specifications mentioned [here](http://support.haptx.com/docs/sdk/page_system_requirements.html) 
## Unity Project
This project includes a simulation to run the HaptX glove and stack blocks with haptic feedback in VR. This project was built on top of the HaptX Starter project, with help from the example made by Argonne National Laboratory. To run this project download the unity project from this repository. Next install unity 2021.3.19f1, steam VR, and any software needed to get your VR headset working. We used a Varjo Aero Headset however any headset that is compatible with Open VR should work. For best resuslts, the HaptX hand configuration shoudl also be used to make a model of the user's hand. Once all of the required software is installed, to run the project begin by setting up the the HTC Vive Base Stations and pairing the tracker on the glove using steam VR. Next open the project using unity hub and run it by pressing the run button at the top of the scene. Once the prject is running, the user shoudl be abel to see the scen which includes 3 blocks, a table and the model of the hand that you have made (if no model is made the default will be used). The  project is designed to track the three blocks (position:x,y,z,angle:quaternion), and the position onf the palm of the hand and each of the finger tips 
## Data Set
## Data Format and cleaning
## Transfomer
