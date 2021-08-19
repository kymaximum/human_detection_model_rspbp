![visual representation](https://user-images.githubusercontent.com/88376943/129695688-9c62fd0c-6cec-4074-8ac4-04c0a1fb2f38.png)
# Human Detection Project Using Raspberry Pi and Databases 

**Background**: 
The purpose of this project is to create a model that can monitor the number of people in a set area over a prolonged period of time. 

This project is relevant because it can be utilized to prevent gatherings with an excessive number of people (that may exacerbate the spread of COVID-19) and can be used to monitor restricted areas even after COVID-19. One possible way to maximize the utility of this project might be to put it in an elevator during COVID-19, since it’s a closed and (often) croweded space.

Previously, other human detection models have been implemented by colleges like Yonsei University. In this project, I focused on creating a model that is lightweight and easy to implement.


**Materials**: Raspberry Pi, Webcam, Monitor


**Steps**: RESEARCH HUMAN DETECTION + YOLOV4 -> YOLOV4TINY + SET UP CAMERA + DOWNLOAD PACKAGES + RUN THE CODE ON RASPBERRY PI USING PUTTY and USE SSH TO CONNECT TO REMOTE SERVER + EDIT CODE TO BETTER APPLY TO PROJECT+ LEARN SQL + LINK YOUR RESULTS TO A SEPARATE DATABASE USING PHPMYADMIN

I researched how I could perform human detection, and decided to use a human detection API (https://github.com/DoranLyong/yolov4-tiny-tflite-for-person-detection) that revolved around Yolov4 (an object detection network). Because I wanted to make this project lightweight, I decided to use a Raspberry Pi as the computing source. The human detection code was unable to run on the Raspberry Pi due to its minimal size, so I researched how I could run the code using Yolov4Tiny. After setting up the camera and running the human detection code on my computer to check if it worked(shown in image), I decided to run the code on Raspberry Pi. ![test_desktop](https://user-images.githubusercontent.com/88376943/130030061-7fa5d7aa-43f8-4311-b9b9-57da1c7b541a.PNG)

While running the code, I discovered that I needed to update or download new versions of packages (ex. tensorflow). Afterwards, I used Putty and SSH to connect from Visual Studio to the remote server. 

Instead of creating a desktop application, I decided to link human detection to a database and monitor the data in real time. Due to this, I edited the code to accommodate for these changes. For instance, I got rid of the code that focused on drawing boxes and the code that focused on display/UI. I added more code that could count the number of total people detected, compare the number to the inputted limit, and give a status report along with time.

Here's an example of the type of code I got rid of:
````
color = colors[int(track.track_id) % len(colors)]
            color = [i * 255 for i in color]
            cv2.rectangle(frame, (int(bbox[0]), int(bbox[1])), (int(bbox[2]), int(bbox[3])), color, 2)
            cv2.rectangle(frame, (int(bbox[0]), int(bbox[1]-30)), (int(bbox[0])+(len(class_name)+len(str(track.track_id)))*17, int(bbox[1])), color, -1)
            cv2.putText(frame, class_name + "-" + str(track.track_id),(int(bbox[0]), int(bbox[1]-10)),0, 0.75, (255,255,255),2)
````
Below is the code for counting the number of people detected and figuring out whether the number exceeds the inputted limit.

````
num_objects = valid_detections.numpy()[0]
[....]
print(num_objects)

if(num_objects < lim_in or num_objects == lim_in):
    status_input = 'good'
if(num_objects == lim_in - 1):
    status_input = 'warning'   
else:
    status_input = 'bad'
````

I also had to learn basic SQL in order to create tables + columns and export data to those tables + columns. Afterwards, I was able to run the program (connected to a webcam) on my Raspberry Pi and link the results in real time to a separate database. The code below illustrates how I was able to send information to a database using SQL and python.

````
import psycopg2
connection = psycopg2.connect(host="192.168.0.12", dbname="example_database", user="postgres", password="example password", port="5432")
cur = connection.cursor()

cur.execute("INSERT INTO public.\"People\"(numppl, lim, status, t) VALUES(%s, %s, %s, %s)", (3, 5, 'good', 4))

connection.commit()
````

**Challenges Faced During Development:**
I faced numerous challenges while developing this project. I often encountered errors due to packages not being updated or not fitting my model. For instance, I had to spend several hours making sure that Tensorflow/Cmake had been installed correctly and were updated versions, because Tensorflow kept on erroring. I eventually overcame this challenge by reinstalling python first, then installing the correct version of tensorflow.

Initially, it was also difficult to connect from the Raspberry Pi to the separate database. Eventually, I realized that this was because the server's port wasn't opening. After this error was rectified, it connected well.

Although learning SQL wasn't too difficult, one challenge I faced afterwards was running SQL code through Python. Fortunately, I eventually figured it out.

**Results**:
- A small webcam connected to a Raspberry Pi sends data (# ppl, limit, time, status) to a separate device (users can monitor the number of ppl from far away).
- My project can detect the number of people in a setting 
- Users can input a desired limit of ppl.
- Status is displayed based on whether the number of ppl. currently in a space is greater than, equal to or less than the limit
