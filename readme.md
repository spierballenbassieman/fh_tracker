De fh_tracker repository is een aanpassing op https://github.com/theAIGuysCode/yolov4-deepsort.

Gebaseerd op paper: https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9739737

ByteTrack is eigenlijk beter dan DeepSort: https://arxiv.org/pdf/2204.06918.pdf#cite.Nillius2006Multi
En StrongSort (Make DeepSort Great Again, 2022) ook. Ik vermoed dat de aanpassing van DeepSort naar StrongSort makkelijk is, aangezien de files erg op elkaar lijken:
https://github.com/mikel-brostrom/Yolov5_StrongSORT_OSNet/tree/master/strong_sort/sort

Nice background information DeepSort: https://programming.vip/docs/deepsort-multi-target-tracking-algorithm.html

# Field hockey player tracker

Hello and welcome to the code for my field hockey player tracker. 
This tracker has 4 stages and the code is organized as follows:
* Detection using Yolo v4. Most of the code for yolo v4 resides in the "core" folder
* Tracking using DeepSORT. The classes for this part can be found in the "deep_sort" folder
* Color detection. I added the code for this into the deep_sort folder and adjusted the deepSORT 
classes to work with it. The main function can be found in color_detect.py
* I created the homography_coordinates.py script to do camera calibration and added it to the main folder.
The rest of the transformation happens inside the object_tracker.py script

# Video input used
De lange filmpjes waren te groot om te uploaden. In de map data zitten wel twee ingekorte fragmenten.

https://pspagnolo.jimdofree.com/download/ of direct: https://www.dropbox.com/s/3zhms9iv4gy3k81/Sequences.zip?dl=1

Ik denk dat ID-3 en ID-4 de beste filmpjes zijn om te testen qua occlusions etc.


# Command used:
python3 object_tracker.py --video /home/basheuver/yolov4-deepsort/data/video/n1.mp4 --output /home/basheuver/yolov4-deepsort/outputs/outpu3ca.avi --model yolov4 --dont_show --jersey_colors 'white','blue','orange' --max_age 95 --nms_overlap 0.7 --n_init 14 --iou 0.8 --cosine 0.2

## Running the tracker

I recommend using Google Colab to run the tracker, since it gives acces to a GPU for free.

1. To run the tracker, first clone this repository by running the following command:
```!git clone https://github.com/S-Hermanides/fh_tracker```

2. Download pre-trained yolov4.weights file [here](https://drive.google.com/open?id=1cewMfusmPjYWbrnuJRuKhPMwRe_b9PaT)
Copy and paste yolov4.weights from your downloads folder into the 'data' folder of this repository.

3. Create the Yolo v4 model by running the following command in the terminal:  
```!python save_model.py --model yolov4```  
This converts the weights into a tensorflow model that's ready to be used

4. Run the tracker with the following command, replacing the parts in <brackets> with your information:  
```!python object_tracker.py --video '<input_video_path> --output <output_video_path> --output_2 <output_map_view_path> --jersey_colors <list_of_colors>```  
There are a number of parameters to tune. Below is a list and an example of how I achieved my final result.

```
# These variables are most likely to be changing

# Input jersey colors to identify teams: Team1, Team2
jersey_colors = 'white','blue'
# I have created a color dictionary in color_detect.py, but values may need to be tuned for your video, lighting etc.

# score threshold is a hyperparameter for Yolo, the confidence score of a detection being a certain class.
score_threshold = 0.5

# Colab can't show video, so set flag to True if you are using a GPU on Google Colab
dont_show_flag = True

# iou threshold is a deepsort hyperparameter, Intersection over Union (predicted versus actual detection box overlap). High value means high chance the boxes are the same detection. 0.45 is default
iou_threshold = 0.45
# max cosine distance is used by deepsort to re-identify detections after losing them (based on the extracted 'appearance' detection vector)
max_cosine_distance = 0.4
# max age is the number of frames that can be "missed" before a unique detection ID is considered dead. Tune if the same object gets multiple IDs. Default is 60
max_age_par = 60
# Number of consecutive detections before the track is confirmed. Default is 3
n_init_par = 3

# If not None, fix samples per class to at most this number. Removes the oldest samples when the budget is reached.
nn_budget = None
# nms is a method to prevent multiple detections for the same object. threshold values often used are 0.3-0.5. Tune if multiple boxes appear
nms_max_overlap = 1.0
```

My own result was achieved by running with these settings:
```!python object_tracker.py --video '/content/drive/My Drive/Colab/project_5/test2.mov' --output '/content/drive/My Drive/Colab/project_5/result2.mp4' --output_2 '/content/drive/My Drive/Colab/project_5/result_bev2.mp4' --output_format mp4v --iou 0.4 --score 0.5 --cosine 0.3 --nms_overlap 0.7 --max_age 60 --n_init 3 --jersey_colors 'white','light blue' --color_threshold 0.08```
