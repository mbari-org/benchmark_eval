Hi! This is elaine's documentation for creating benchmark datasets and evaluating models using HOTA :)


## How to annotate a video in Rectlabel

To annotate, need img, xml, and label folders. 
- Go to File -> Convert video to image frames (imgsize 1920, 60 fps). Put frames into an img folder. 
- Create blank xml folder. 
- Run **model + tracker** cell in **tracker_output.ipynb** file to get yolo labels. In rectlabel, go to export -> import yolo txt files. Select label folder for yolo txt files and xml file for where annotations go. 

Helpful rectlabel tips
- Use move to edit boxes and create to create boxes
- Turn on "Show labels on boxes" and "Show checkboxes on labels table"
- Double click bounding box to edit
- Command + arrow key to move through frames faster

## How to get Ground Truth files
- After cleaning up detections in rectlabel, go to Export -> Export yolo txt files, save them in a separate folder. 
- Run the **ground truth** cell in tracker_output.ipynb to add the track id to the yolo txt files. Make sure to edit the paths accordingly. 
- Run the **convert format** cell in tracker_output.ipynb to convert from yolo to motchallenge format. Make sure to edit the paths accordingly. The final ground truth file should end up in *benchmark_eval/data/gt/mot_challenge/MOT17-train/insert_vidseq_name/gt*

## How to get Model Output files
- Run the **model + tracker cell** in the **tracker_output.ipynb** file to get the model detections. 
- Run the convert format cell to convert detections from yolo to motchallenge format. Make sure to edit the paths accordingly. The final txt file should end up in *benchmark_eval/data/trackers/mot_challenge/MOT17-train/YOLOTracker*

## How to get HOTA scores
- In the video sequence folders under *benchmark_eval/data/gt/mot_challenge/MOT17-train*, add a folder called img1 to hold all the video image frames (should be the original frames, no bounding boxes).
- Create a seqinfo.ini file, which should be in the format below. The name should match the video sequence folder name, and the imDir should match the image directory name. Adjust frame rate, sequence length, image width, and image height accordingly. 
```
[Sequence]
name= insert_vid_seq_name
imDir=img1
frameRate=30
seqLength=600
imWidth=1920
imHeight=1080
```
- Next, edit the vidseq_names.txt under *benchmark_eval/data/gt/mot_challenge/seqmaps*. The format should have “name” at the top, and then all the video sequences you would like to get the HOTA values for. 
In the terminal, make sure you are in the benchmark_eval directory and type in this command:
```
python scripts/run_mot_challenge.py \                                                                     
--GT_FOLDER data/gt/mot_challenge \
--TRACKERS_FOLDER data/trackers/mot_challenge \
--SEQMAP_FILE data/gt/mot_challenge/seqmaps/vidseq_names.txt \
--SPLIT_TO_EVAL train \
--BENCHMARK MOT17 \
--TRACKER_SUB_FOLDER ' '
 ```
