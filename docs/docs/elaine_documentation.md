Hi! This is elaine's documentation for creating benchmark datasets and evaluating models using HOTA :)

**insert a link to github codebase**

A majority of the code in this repository is the TrackEval github repo. I added a bit of code here and there and set up the file structure to run the HOTA metric for our benchmark dataset and renamed it benchmark-eval. 

# Installation
<br>
## Setting Up The Environment

First, install Python 3.10.13

```
pyenv install 3.10.13
```

Then, create a new virtual environment to work in
```
pyenv virtualenv 3.10.13 env-name
```

Lastly, install the benchmark_eval_requirements.txt file
```
pip install -r requirements.txt
```

*You'll see that there is also a requirements.txt file in the repository. That was from the original TrackEval, but it's a bit outdated and doesn't have updated installations. 

<br>
# Usage 
<br>
## How to annotate a video in Rectlabel

To annotate, need img, xml, and label folders. 

- Go to File -> Convert video to image frames and put the resulting frames into an img folder. 
- Create blank xml folder. 
- Run **model + tracker** cell in **tracker_output.ipynb** file to get yolo labels. In rectlabel, go to export -> import yolo txt files. Select label folder for yolo txt files and xml file for where annotations go. 

Helpful rectlabel tips

- Use move to edit boxes and create to create boxes
- Turn on "Show labels on boxes" and "Show checkboxes on labels table"
- Double click bounding box to edit
- Command + arrow key to move through frames faster

<br>
## How to get Ground Truth files
- After cleaning up detections in rectlabel, go to Export -> Export yolo txt files, save them under *benchmark_eval/temp*.
- Under *benchmark_eval/data/gt/mot_challenge/MOT17-train*, create a folder named after your vidseq_name and create an empty folder named gt. ie (../simple_mid/gt).
- Go to tracker_output.ipynb and run the **setting variables cell**. Make sure that the frame dimension and video sequence name are correct. 
- Run the **ground truth cell** in tracker_output.ipynb to add the track id to the yolo txt files. This will also convert the file from yolo to motchallenge format. The final ground truth file should end up in *benchmark_eval/data/gt/mot_challenge/MOT17-train/insert_vidseq_name/gt*

#### What is the MOTChallenge format?

The MOTChallenge file format is required for the metric calculation in TrackEval. The files from rectlabel are exported as yolo txt files, so they need to be converted. 

MOTChallenge format:

<frame\>, <id\>, <x\>, <y\>, <w\>, <h\>, <conf\>, <class\>, <vis\>

Example line:

1, 1, 1002.00, 610.50, 45.00, 58.50, 1, 1, -1

This means that in frame 1, an object with a track id 1 has a bounding box with a top left corner at (1002.00, 610.50) and a width of 45.00 and height of 58.50. Confidence is always set to 1. Class and vis (visibility) are not required by TrackEval, so I set them manually to 1 and -1, respectively. 

<br>
## How to get Model Output files
- Go to tracker_output.ipynb and double check the **setting variables cell**.
- Run the **model + tracker cell** to get the model detections and convert them from yolo to motchallenge format. The final txt file should end up in *benchmark_eval/data/trackers/mot_challenge/MOT17-train/YOLOTracker*

*if an error occurs, double check the paths. There are folders being generated and folders that you add yourself, so its possible that things can get mixed up!

<br>
## How to get HOTA scores
- In the video sequence folders under *benchmark_eval/data/gt/mot_challenge/MOT17-train*, add a folder named after your video sequence. In this folder:
    - add a folder called img1 to hold all the video image frames (should be the original frames, no bounding boxes).
    - Create a seqinfo.ini file - the format is shown under "Example Files". The name should match the video sequence folder name, and the imDir should match the image directory name. Adjust frame rate, sequence length, image width, and image height accordingly. 
    - There should already be a gt folder from the "How to get Ground Truth files" step. 
- Next, edit the vidseq_names.txt under *benchmark_eval/data/gt/mot_challenge/seqmaps*. The format should have “name” at the top, and then all the video sequences you would like to get the HOTA values for. 
- Lastly, in the terminal, make sure you are in the benchmark_eval directory and type in this command:

```
python scripts/run_mot_challenge.py \                                                                     
--GT_FOLDER data/gt/mot_challenge \
--TRACKERS_FOLDER data/trackers/mot_challenge \
--SEQMAP_FILE data/gt/mot_challenge/seqmaps/vidseq_names.txt \
--TRACKER_SUB_FOLDER ' '
```

#### What is HOTA?

Plain english: metric that measures how well trajectories of detections align, averaged over all trajectories, while also taking into account incorrect detections. 

- balances association and detection. Previously used MOTA score was biased toward detection and IDF1 score was biased toward association.
- Made up of 6 secondary metrics:
  - **DetA (detection accuracy)** - percentage of aligning detections
  - **AssA (association accuracy)** - average alignment between trajectories, averaged over all detections. Basically how well tracker maintains identity consistency
  - **DetRe (detection recall)** - how well tracker finds all ground truth detections
  - **DetPr (detection precision)** - how well tracker manages to NOT predict extra detections that arent there
  - **AssRe (association recall)** - how well trackers can avoid splitting same object into multiple shorter tracks
  - **AssPr (association precision)** - how well trackers avoid merging multiple objects into a single track
  - **LocA (localization accuracy)** - measures spatial accuracy (IoU) between gt/predicted bounding boxes

<br>
# Helpful Tips

<br>
## Directory Structure

This is a more in depth explanation of the parts of the directory I wanted to highlight. 

Look at the simple_mid folder under *data/gt/mot_challenge/MOT17-train*. There are 3 items:

- **gt folder**: holds gt.txt, which contains ground truth values. The file should be in motchallenge format. 
- **img1 folder**: Holds the image frames of the video sequence. Should be taken straight from original video, no bounding boxes. Current videos are 10 seconds at 60 fps, so there are 600 images in each img1 folder. 
- **seqinfo.ini file**: holds video sequence information such as name, image directory name, frame rate, etc. 

Now look at the seqmaps folder under *data/gt/mot_challenge*. It contains:

- **vidseq_names.txt**: file specifying all video sequences you would like to include when running the HOTA metric. 

Lastly, look under *data/trackers/mot_challenge/MOT17-train/YOLOTracker*. It contains:

- **simple_mid.txt**: file containing model output values. The file should be in motchallenge format. This will be directly compared with the gt.txt file. 

```
benchmark_eval   
│
└───data
    └───gt
    |   └───mot_challenge
    |       └───MOT17-train
    |       |   └───simple_mid
    |       |       └───gt    
    |       |       |       gt.txt
    |       |       └───img1
    |       |       |   |   simple_mid_1.jpg
    |       |       |   |   simple_mid_2.jpg
    |       |       |   |   ...
    |       |       |
    |       |       └───seqinfo.ini
    |       └───seqmaps
    |           └───vidseq_names.txt
    └───trackers
        └───mot_challenge
            └───MOT17-train
                └───YOLOTracker
                    └───simple_mid.txt

```
Why are there so many folders and why are their names so specific? Honestly, I don't know. The track-eval github I pulled this from had little documentation and I added folders until it worked. 

<br>
## Example Files

Here are some examples of what the contents of files should look like:

seqinfo.ini file:

```
[Sequence]
name= insert_vid_seq_name
imDir=img1
frameRate=60
seqLength=600
imWidth=1920
imHeight=1080
```

vidseq_names.txt file:

*make sure to include the word "name" at the top before listing the video sequences!

```
name
simple_mid
simple_ben
difficult_mid
difficult_ben
```

First 5 lines of simple_mid.txt:
```
1, 1, 1002.00, 610.50, 45.00, 58.50, 1, 1, -1
1, 2, 41.91, 207.94, 110.06, 67.13, 1, 1, -1
1, 3, 1134.75, 567.75, 42.00, 28.50, 1, 1, -1
10, 1, 1006.06, 614.45, 44.20, 56.91, 1, 1, -1
10, 2, 33.26, 200.92, 105.12, 64.41, 1, 1, -1
```

And the first 5 lines of its corresponding gt.txt:
```
1, 8, 441.00, 632.00, 21.00, 31.00, 1, 1, -1
1, 1, 1002.00, 610.50, 45.00, 58.50, 1, 1, -1
1, 2, 41.91, 207.94, 110.06, 67.13, 1, 1, -1
1, 3, 1134.75, 567.75, 42.00, 28.50, 1, 1, -1
10, 8, 431.00, 635.00, 21.00, 31.00, 1, 1, -1
```
