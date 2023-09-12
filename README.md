# 3D-Gaussian_Splatting

What are 3D Gaussian Splats?
At a high level, 3D Gaussian splats, like NeRFs or photogrammetry methods, are a way to create a 3D scene using a set of 2D images. Practically, this means that all you need is a video or a set of photos of a scene, to obtain a 3D representation of it — enabling you to reshoot it, or render it from any angle.

Step-by-step Tutorial:

first of all Cloning the Repository: 

# HTTPS
	git clone https://github.com/graphdeco-inria/gaussian-splatting --recursive

Hardware Requirements : CUDA-ready GPU with Compute Capability 7.0+
			24 GB VRAM (to train to paper evaluation quality)
			Please see FAQ for smaller VRAM configurations
	

Software Requirements: 	Conda (recommended for easy setup)
			C++ Compiler for PyTorch extensions (we used Visual Studio 2019 for Windows)
			CUDA SDK 11 for PyTorch extensions, install after Visual Studio (we used 11.8, known issues with 11.6)
			C++ Compiler and CUDA SDK must be compatible							
# Setup
-> Do local Setup using following commands :

	SET DISTUTILS_USE_SDK=1 # Windows only
	conda env create --file environment.yml
	conda activate gaussian_splatting

--> Please note that this process assumes that you have CUDA SDK 11 installed, not 12. For modifications, see below.

# Running 
-> To run the optimizer, simply use following commands:
	
	python train.py -s <path to COLMAP or NeRF Synthetic dataset>

<details><summary>Command Line Arguments for train.py</summary> 

--source_path / -s : Path to the source directory containing a COLMAP or Synthetic NeRF data set.
--model_path / -m  : Path where the trained model should be stored (output/<random> by default).
--images / -i      : Alternative subdirectory for COLMAP images (images by default).
--eval		   : Add this flag to use a MipNeRF360-style training/test split for evaluation.
</details>
# Evaluation

--> By default, the trained models use all available images in the dataset. To train them while withholding a test set for evaluation, use the --eval flag.     
--> This way, you can render training/test sets and produce error metrics as follows:

	python train.py -s <path to COLMAP or NeRF Synthetic dataset> --eval # Train with train/test split
	python render.py -m <path to trained model> # Generate renderings
	python metrics.py -m <path to trained model> # Compute error metrics on renderings

--> Command Line Arguments for render.py : 
			
			--model_path / -m  : Path to the trained model directory you want to create renderings for.
			--source_path / -s : Path to the source directory containing a COLMAP or Synthetic NeRF data set.
			--images / -i      : Alternative subdirectory for COLMAP images (images by default).
			--resolution / -r  : Changes the resolution of the loaded images before training. If provided 1, 2, 4 or 8, uses original, 1/2, 1/4  					     or 1/8 resolution, respectively. For all other values, rescales the width to the given number while maintaining 					     image aspect. 1 by default.
--> Command Line Arguments for metrics.py : 
	
			--model_paths / -m : Space-separated list of model paths for which metrics should be computed.

# Interactive Viewers
 
--> Our viewing solutions are based on the SIBR framework, developed by the GRAPHDECO group for several novel-view synthesis projects.

--> Installation from Source : If you cloned with submodules (e.g., using --recursive), the source code for the viewers is found in SIBR_viewers. The network 			       viewer runs within the SIBR framework for Image-based Rendering applications.	

# For Windows, simply use following commands:

		cd SIBR_viewers
		cmake -Bbuild .
		cmake --build build --target install --config RelWithDebInfo

# For Ubuntu 22.04, simply use following commands:

# Dependencies

	sudo apt install -y libglew-dev libassimp-dev libboost-all-dev libgtk-3-dev libopencv-dev libglfw3-dev libavdevice-dev libavcodec-dev libeigen3-dev 	libxxf86vm-dev libembree-dev

# Project setup
		cd SIBR_viewers
		cmake -Bbuild . -DCMAKE_BUILD_TYPE=Release # add -G Ninja to build faster
		cmake --build build -j24 --target install


# Navigation in SIBR Viewers: 

--> The SIBR interface provides several methods of navigating the scene. By default, you will be started with an FPS navigator, which you can control with W,           A, S, D, Q, E for camera translation and I, K, J, L, U, O for rotation. 

--> Alternatively, you may want to use a Trackball-style navigator (select from the floating menu). You can also snap to a camera from the data set with the     Snap to button or find the closest camera with Snap to closest. 

--> The floating menues also allow you to change the navigation speed. You can use the Scaling  Modifier to control the size of the displayed Gaussians, or show the initial point cloud.

# Running the Network Viewer : 

--> After extracting or installing the viewers, you may run the compiled SIBR_remoteGaussian_app[_config] app in <SIBR install dir>/bin, 

e.g.: ./<SIBR install dir>/bin/SIBR_remoteGaussian_app

# Running the Real-Time Viewer : 

--> After extracting or installing the viewers, you may run the compiled SIBR_gaussianViewer_app[_config] app in <SIBR install dir>/bin,

e.g.: ./<SIBR install dir>/bin/SIBR_gaussianViewer_app -m <path to trained model>


# How to train your own models? (Tutorial)

Important: before starting, Make sure that You have completed the above steps. 
--> In particular, this will require a CUDA-ready GPU with 24 GB of VRAM.

Step 1: Record the scene

Recording the scene is one of the most important steps because that's what the model will be trained on. You can either record a video (and extract the frames afterwards) or take individual photos. Be sure to move around the scene, and to capture it from different angles. Generally, the more images you have, the better the model will be. A few tips to keep in mind to get the best results:

--> Avoid moving too fast, as it can cause blurry frames (which 3D Gaussian Splats will try to reproduce)
--> Try to aim for 200-1000 images. Less than 200 images will result in a low quality model, and more than 1000 images will take a long time to process in step 2.
--> Lock the exposure of your camera. If it's not consistent between frames, it will cause flickering in the final model.

Once you're done. Place your images in a folder called input, like this:

📦 $FOLDER_PATH
 ┣ 📂 input
 ┃ ┣ 📜 000000.jpg
 ┃ ┣ 📜 000001.jpg
 ┃ ┣ 📜 ...

Step 2: Obtain Camera poses

Obtaining camera poses is probably to most finicky step of the entire process, for inexperienced users. The goal is to obtain the position and orientation of the camera for each frame. This is called the camera pose. There are several ways to do so:

-> Use COLMAP. COLMAP is a free and open-source Structure-from-Motion (SfM) software. It will take your images as input, and output the camera poses. It comes 
with a GUI and is available on Windows, Mac, and Linux.

# Installation of colmap : https://colmap.github.io/install.html

# After installed the colmap,

--> Open COLMAP GUI. On linux, you can run colmap gui in a terminal. On Windows and Mac, you can open the COLMAPapplication.

--> Start a new project by clicking on File > New project. A dialog opens up. Create a new database by clicking on New and calling it database.db. 
--> For Images, select the folder where your images are located. Then click on Save.

--> Run Feature extraction by clicking on Processing > Feature extraction. Keep most parameters as default. Check the "Shared for all images" options (if you didn't zoom in or out between frames), and set first_octave to 0 (this will be faster than with the default -1). Then click on Extract (this will take a few seconds).

--> Run Feature matching by clicking on Processing > Feature matching. Because you have recorded the object by walking around it, neighboring images should be close in space, so you can use the Sequential matching mode. This will be faster than the default Exhaustive mode (if reconstruction fails, you can try again with Exhaustive). Then click on Run (this will take a few seconds to a minute).

--> Now comes the last step: Reconstruction. This will take the longest (from a few minutes to a few hours depending on the number of images). First, click on Reconstruction > Reconstruction options. Uncheck multiple_models (since we're reconstructing a single scene) and close the window. Then start the reconstruction optimization by clicking on Reconstruction > Start reconstruction.

-->Once COLMAP has finished, you will see the camera poses in the GUI (in red) along with a sparse pointcloud of the scene. Now export the camera poses by clicking on File > Export model and save it in a folder distorted at the same level as the input folder. You can now close COLMAP.

# The folder structure of your model dataset should now look like this:

					       📦 $FOLDER_PATH
 ┣ 														 |_	📂 input
 ┣ 														 |_	📂 distorted
 ┃ ┣ 														 |_	📂 0
 ┃ ┃ 															┣ |_📜 points3D.bin
 ┃ ┃ ┣ 														 |_📜 images.bin
 ┃ ┃ ┗ 														 |_📜 cameras.bin

# run the following script:
				python convert.py -s $FOLDER_PATH --skip_matching

--> The folder structure of your model dataset should now look like this:

📦 $FOLDER_PATH
 ┣ 📂 (input)
 ┣ 📂 (distorted)
 ┣ 📂 images
 ┣ 📂 sparse
 ┃ ┣ 📂 0
 ┃ ┃ ┣ 📜 points3D.bin
 ┃ ┃ ┣ 📜 images.bin
 ┃ ┃ ┗ 📜 cameras.bin

Step 3: Train the 3D Gaussian Splatting model

				python train.py -s $FOLDER_PATH -m $FOLDER_PATH/output

-->This will save the model in the $FOLDER_PATH/output folder.

Step 4: Visualize the model
 
-->The folder structure of your model dataset should now look like this:

📦 $FOLDER_PATH
 ┣ 📂 images
 ┣ 📂 sparse
 ┣ 📂 output
 ┃ ┣ 📜 cameras.json
 ┃ ┣ 📜 cfg_args
 ┃ ┗ 📜 input.ply
 ┃ ┣ 📂 point_cloud
 ┃ ┃ ┣ 📂 iteration_7000
 ┃ ┃ ┃ ┗ 📜 point_cloud.ply
 ┃ ┃ ┣ 📂 iteration_30000
 ┃ ┃ ┃ ┗ 📜 point_cloud.ply


--> Once installed, find the SIBR_gaussianViewer_app binary and run it with the path to the model as argument:

						SIBR_gaussianViewer_app -m $FOLDER_PATH/output


## You get a beautiful visualizer of your trained model! Make sure to select Trackball mode for a better interactive experience.














