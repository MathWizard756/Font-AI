# Font AI

 An AI created to recognise characters of different fonts> 

https://upload.wikimedia.org/wikipedia/commons/thumb/1/1e/Abecedarium.png/1200px-Abecedarium.png

## How it Works

By training the resnet-18 network and using the Imagenet software, the computer runs through hundreds and thousands of images, being able to recognise the structure of letters. attempting to give the computer vision, just like a human.

## Running this project

First and formost, you will need a Jetson-nano by Nvidia to follow along with this project. In addition, you want to know how to set the nano up in headless mode, as that is what I used for this project. Here is a link for a headless mode guide:
https://www.youtube.com/watch?v=FyrXnNpPR6s

Now, let's get started: 

- Log into your nano 

ssh <name>@192.168.55.1

- Run these commands to make sure git and cmake are installed. You will be prompted to enter your password

sudo apt-get update
sudo apt-get install git cmake

- Clone the jetson-inference project

git clone --recursive https://github.com/dusty-nv/jetson-inference
cd jetson-inference
git submodule update --init

- In order to install the python packages, run this command

sudo apt-get install libpython3-dev python3-numpy

- Next, make a build directory to build your project into. Make sure you are in the jetson-inference directory before running these commands

mkdir build

- Change directories into build and run make to build the project

cd build
cmake ../

- While configuring, you will see the downloader tool. You can access this tool at any time using commands, so for now, just make sure you are downloading resnet and googlenet

- Once this finishes loading you should see a space to run commands again. Make sure that you are still in the build directory and run these commands

make
sudo make install
sudo ldconfig

- Now you will be able to run the googlnet and resnet-18 network for future steps 

- We will now retrain the image.net program to include classifying fonts 

- Now download this dataset onto your computer (not nano). Here is the link to the page: https://www.kaggle.com/datasets/thomasqazwsxedc/alphabet-characters-fonts-dataset/code?resource=download

- Then open up the terminal on your computer and type in this command to move the file on your computer onto your nano

scp C:\Users\<computer name>\downloads\archive.zip nvidia@192.168.55.1:/home/nvidia/jetson-inference/python/training/classification/data

- Log onto the nano and change directories into jetson-inference/python/training/classification/data

cd jetson-inference/python/training/classification/data 

- Now we have to create a new folder called "font_catalog"  inside the data directory 

mkdir font_catalog

- Run this command to unzip the file inside the new folder

unzip archive.zip -d font_catalog

- This database is not sorted into val, teach, and test directories, therefore we need to create them ourselves. Create three directories inside the font_catalog directory called val, train, and test 

mkdir val 
mkdir train 
mkdir test

- Go to the second Images directory 

cd Images/Images

- From this folder, three seperate times, copy the second Images directory into the val, train, and test folders respectively

cp -r /home/<username>/python/training/classification/data/font_catalog/Images/Images /home/<username>/python/training/classification/data/font_catalog/train

cp -r /home/<username>/python/training/classification/data/font_catalog/Images/Images /home/<username>/python/training/classification/data/font_catalog/test

cp -r /home/<username>/python/training/classification/data/font_catalog/Images/Images /home/<username>/python/training/classification/data/font_catalog/val

- Then move all of the files one directory out for each folder, and delete the Images directory once the images are moved out 

mv /home/<username>/python/training/classification/data/font_catalog/<train + val + test>/Images/<A.............Z> ..

rm -r Images

- From the jetson-inference folder, run ./docker/run.sh to run the docker container.

./docker/run.sh

- Change directories so you are in jetson-inference/python/training/classification

cd python/training/classification

- Run the training script to re-train the network where the model-dir argument is where the model should be saved and where the data is.  You should immediately start to see output, but it will take a very long time to finish running.

python3 train.py --model-dir=models/font_catalog data/font_catalog 

- While it is running, you can stop it at any time using "Ctl+C". You can also restart the training again later using the "--resume" and "--epoch-start" flags, so you do not need to wait for training to complete before testing out the model.

- Run python3 train.py "--help" for more information about each option that is available for you to use, including other networks that you can try with the "--arch" flag


- Make sure you are in the docker container and in jetson-inference/python/training/classification, then run the onnx export script 

python3 onnx_export.py --model-dir=models/font_catalog

- Look in jetson-inference/python/training/classification/models/font_catalog to see if there is a new model called resnet18.onnx there. That is your re-trained model!


- In order to see how your network functions you can run images through them. You can use the imageNet command line arguments to test out your re-trained network.

- Exit the docker container by pressing Ctl + D.

- On your nano, ensure you are in jetson-inference/python/training/classification.

- Use ls models/font_catalog/ to make sure that the model is on the nano. You should see a file called resnet18.onnx.

ls models/font_catalog/

- Set the NET and DATASET variables

NET=models/font_catalog
DATASET=data/font_catalog

- Go into the font_catalog directory and create a text file called "labels.txt"

cat > labels.txt

- Edit the txt file and write the name of the letters of the alphabet (your image folders) capitalized, one per line 

nano labels.txt 

  A
  B
  C
  D 
  .
  .
  .
  .
  Z

- Congrats! The computer has been trained! Now it is time to try to test it out 

- Run this command to see how it operates on an image from the font_catalog folder 

imagenet.py --model=$NET/resnet18.onnx --input_blob=input_0 --output_blob=output_0 --labels=$DATASET/labels.txt $DATASET/test/A/0.png 

- Now let the computer analyze what the font is. Congrats! You are finished.





[View a video explanation here](video link)
