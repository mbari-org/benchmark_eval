# Usage 

## How to Annotate a Video in RectLabel Pro

The benchmark videos are too large to push to GitHub, so they are not included in the repository. They are available on Hugging Face [https://huggingface.co/datasets/MBARI-org/DeepSea-MOT](https://huggingface.co/datasets/MBARI-org/DeepSea-MOT). If you have the videos, they should be located in a `videos` folder under `benchmark_eval`. If you don't have it, create an empty `videos` folder. Inside `videos`, create a folder for your video sequence and insert your video file here. To make a benchmark video out of a video sequence, you will need `images`, `labels`, and `xml` folders. 

1. In RectLabel, select **File -> Convert video to image frames** and put the resulting frames into an `images` folder. 
2. Create an empty `xml` folder. 
3. Go to `tracker_output.ipynb` and set the video sequence, model, and tracker you are using in the **Setting Variables cell**. If you would like to edit the video paths or add new videos, edit the paths here. Run the **Defining Functions cell**.
4. Run **Model + Tracker** cell to get YOLO labels. The model parameters can be adjusted here, e.g. `conf`, `iou`, and `imgsz`. If you are using Windows/Linux, change `device='mps'` to `device='cpu'` or `device='cuda'` (NVIDIA GPU). Tracker parameters can also be adjusted in the yaml file. Copy the `labels` folder from `/runs/detect/track/` directory into your `videos` folder which has `images` and `xml` folders.
5. In RectLabel, select "Open folder", select the `images` folder for the "Images folder" and the empty `xml` folder for the "Annotations folder" (Label format is PASCAL xml, sort images Numeric), then select 'OK'.
6. Select Settings, go to Projects tab, create a new project with the `+`, and make Primary with checkbox. Objects can also be imported to this project with the **Export -> Import object names -> from a file** (e.g. a model names file), and add "track id" to each object in the Settings -> Objects tab, Attributes field by selecting the checkbox.
7. Now go to **Export -> Import YOLO txt files**. Browse to your `labels` folder and select "Open".
8. Edit boxes and labels as needed to create a groundtruth video.
9. To save, select **File -> Save**, or in Settings, in the Label Fast tab, choose checkbox 'Autosave'.
10. Select **File -> Close** folder when completed.

Your file structure could look something like this:

```
videos  
│
└───simple_mid
|    └───Midwater_simple_V4455_20221130T192123Z_prores.mov
|    └───images
|    └───labels  
|    └───xml
| 
└───simple_ben
| ...
```

!!! tip "RectLabel Tips"
    - Use **move** to edit boxes and **create** to create boxes
    - Turn on "Show labels on boxes" and "Show checkboxes on labels table"
    - Double click a bounding box to edit it
    - ++command+left++ / ++command+right++ to move through frames faster

## How to get Ground Truth files

The `data` folder is also not included in GitHub, it is available on Hugging Face [https://huggingface.co/datasets/MBARI-org/DeepSea-MOT](https://huggingface.co/datasets/MBARI-org/DeepSea-MOT). If you have the `data` folder, place it under benchmark_eval. If not, create an empty `data` folder with an empty `gt` folder inside. In here, create a folder named after your `vidseq_name` and create another empty folder named `gt` inside (e.g., `benchmark_eval/data/gt/simple_mid/gt`).

1. After you have cleaned up detections in RectLabel, go to **Export -> Export YOLO txt files**, and save them under the `temp` folder in `benchmark_eval`. You can delete the names file that RectLabel exports to temp folder (e.g., `mbari452k.yaml`).
2. Go to `tracker_output.ipynb` and make sure you have the correct video sequence in the **Setting Variables cell**.
3. Run the **Ground Truth cell** to add the track id to the YOLO txt files and generate `gt.txt file`. This will look for the "labels" folder you exported to `benchmark_eval/temp` and create a folder called `labels_w_trackid`. This folder will then be converted from YOLO to MOT Challenge format. The final ground truth file will be called `gt.txt` and the script will place it under `benchmark_eval/data/gt/{vid_seq_name}/gt/gt.txt`.

### What is the MOT Challenge format?

The MOT Challenge file format is required for the metric calculation in TrackEval. The files from RectLabel are exported as YOLO txt files, so they need to be converted. 

MOT Challenge format:

```
<frame>, <id>, <x>, <y>, <w>, <h>, <conf>, <class>, <vis>
```

Example line:

```
1, 1, 1002.00, 610.50, 45.00, 58.50, 1, 1, -1
```

This means that in frame 1, an object with a track id 1 has a bounding box with a top left corner at (1002.00, 610.50) and a width of 45.00 and height of 58.50. Trackeval does not require confidence, class, or vis (visibility), so for these purposes I set them manually to 1, 1 and -1, respectively. 


## How to get Model Output files
1. Go to `tracker_output.ipynb` and double check the video sequence, model, and tracker in the **Setting Variables cell**.
2. Run the **Model + Tracker cell** to get the model detections and convert them from YOLO to MOT Challenge format. The final txt file should end up in `benchmark_eval/data/predictions/mot_challenge`

!!! note
    The `runs` folder accumulates results from Ultralytics detection & tracking. These files can be cleared out after the evaluation process is complete if you want to save space.

!!! note
    If an error occurs, double check the paths. There are folders being generated and folders that you add yourself, so it's possible paths were not correctly identified.

## How to get HOTA scores
1. If you look under `benchmark_eval/data/gt`, there should be a folder for your video sequence from the "How to get Ground Truth files" step. In this folder:
    a. Add a folder called `img1` to hold all the video image frames (should be the original frames from `images` folder in RectLabel step 1 above, no bounding boxes).
    b. Create a `seqinfo.ini` file - the format is shown in [Tips](tips.md). The name should match the video sequence folder name, and the `imDir` should match the image directory name. Adjust frame rate, sequence length, image width, and image height accordingly. 
    c. There should already be a `gt` folder from the "How to get Ground Truth files" step. 
2. Next, edit the `vidseq_names.txt` under `benchmark_eval/data/seqmaps`. The format should have "name" at the top, and then all the video sequences you would like to get the HOTA values for. An example can be seen in [Tips](tips.md).
3. Lastly, in the terminal, make sure you are in the `benchmark_eval` directory with your pyenv activated, and type in this command:

```bash
python scripts/run_mot_challenge.py  \
  --GT_FOLDER data/gt \
  --TRACKERS_FOLDER data/predictions \
  --SEQMAP_FILE data/seqmaps/vidseq_names.txt \
  --TRACKER_SUB_FOLDER ''
```

---

!!! question "What is HOTA?"

    Plain english: [**HOTA**](https://link.springer.com/article/10.1007/s11263-020-01375-2) (higher order tracking accuracy) is a metric that measures how well trajectories of detections align, averaged over all trajectories, while also taking into account incorrect detections. 

    - Balances detection, localization, and association. Previously used **MOTA** score was biased toward detection and **IDF1** score was biased toward association.
    - Made up of 6 secondary metrics:
        - **DetA (detection accuracy)** - percentage of aligning detections
        - **AssA (association accuracy)** - average alignment between trajectories, averaged over all detections. Basically how well tracker maintains identity consistency
        - **DetRe (detection recall)** - how well tracker finds all ground truth detections
        - **DetPr (detection precision)** - how well tracker manages to NOT predict extra detections that arent there
        - **AssRe (association recall)** - how well trackers can avoid splitting same object into multiple shorter tracks
        - **AssPr (association precision)** - how well trackers avoid merging multiple objects into a single track
        - **LocA (localization accuracy)** - measures spatial accuracy (IoU) between gt/predicted bounding boxes
