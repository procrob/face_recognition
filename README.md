The face_recognition ROS package provides an simple actionlib server interface for performing different face recognition functionalities in video stream.

********************************************************************************
FaceRecognitionGoal message includes 2 fields: an order_id which is an integer specifying the goal and an order_argument which is a string specifying an argument for the goal if necessary. The face recognition actionlib server accepts 4 different goals:

order_id = 0                                     // (recognize_once) Goal is to acknowledge the first face recognized in the video stream. When the first face is recognized with a confidence value higher than the desirable confidence threshold, the name of the person and the confidence value are sent back to the client as result.

order_id = 1                                     // (recognize_continuous) Goal is to continuously recognize faces in the video stream. Every face recognized with confidence value higher than the desirable confidence threshold and its confidence value are sent back to the client as feedback. This goal is persuaded for infinite time until it is canceled or preempted by another goal.

order_id = 2 AND order_argument = person_name    // (add_face_images) Goal is to acquire training images for a NEW person. The video stream is processed for detecting a face which is saved and used as a training image  for the new person. This process is continued until the desired number of training images for the new person is acquired. The name of the new person is provided as "order_argument"

order_id = 3                                     // (train) The database is (re)trained from the training images

order_id = 4                                     // (exit) The program exits.

*********************************************************************************
Subscribed Topic:

"/camera/image_raw"                              // video stream (standard ROS image transport)

*********************************************************************************
Parameters:

"confidence_value" (double, default = 0.88) : a face recognized with confidence value higher than the "confidence_value" threshold is accepted as valid.
    
"show_screen_flag" (boolean, default = true) : if output screen is shown

"add_face_number" (int, default = 25): a parameter for the "add_face_images" goal which determines the number of training images for a new person to be acquired from the video stream 

*********************************************************************************
How The Face Recognition Works:

Details in http://www.shervinemami.info/faceRecognition.html

The name of persons and the images of their faces to train from are stored in the "train.txt" file.
This file should be in the same directory as where you execute the program. See the example train.txt file for the desirable format of the train.txt (Note: the person numbers start from 1, spaces or special characters are not allowed in persons' names). 
The program trains from the train.txt file and stores the eigenfaces data in "facedata.xml" file.

When the face_recognition actionlib server is executed (Fserver), the program first tries to load data from the facedata.xml file if exists, if not it will try to learn from training images stored in train.txt if any. You can always add training data directly from the video stream using the "add_face_images" goal (order_id = 2).
Note: when program retrains (order_id = 3), the content of facedata.xml is disregarded and the program trains only based on the train.txt file.

**********************************************************************************
Tutorial: 
For demonstration purposes an actionlib client example for the face_recognition simple actionlib server has been provided. 
The client subscribes to face_recognition::FRClientGoal messages. Each FRClientGoal message contains an order_id and an order_argument which specify a goal to be executed by the face_recognition server. After receiving a message, the client sends the corresponding goal to the server. By registering relevant call back functions, the client receives feedback and result information from the execution of goals in the server and prints such information on the terminal. 

1)run "roscore"
2)in separate terminal publish a video stream on topic /camera/image_raw. For example you can use gscam to publish images from your web cam as follows:
  2-1)install gscam package
  2-2)roscd gscam/bin
  2-3)rosrun gscam gscam /gscam/image_raw:=/camera/image_raw
3)in separate terminals run the face recognition server and client as follows:
  3-1)rosrun face_recognition Fserver
  3-2)rosrun face_recognition Fclient
4)in another terminal publish messages on topic /fr_order to test different face recognition functionalities as follows: (notice the info which are printed on the client terminal after each command) 
  4-1) rostopic pub -1 /fr_order face_recognition/FRClientGoal -- 2 "your_name"             //to acquire training images for your face: you should try to appear in the video stream!
  4-2) rostopic pub -1 /fr_order face_recognition/FRClientGoal -- 1 "none"                  //to recognize faces continuously. This would not stop until you preempt or cancel the goal. So lets preempt it by sending the next goal.
  4-3) rostopic pub -1 /fr_order face_recognition/FRClientGoal -- 2 "your_friend's_name"    //add training images for a new person
  4-4) rostopic pub -1 /fr_order face_recognition/FRClientGoal -- 0 "none"                  //recognize once
  4-5) rostopic pub -1 /fr_order face_recognition/FRClientGoal -- 0 "none"                  //recognize continuous
  4-6) rostopic pub -1 /fr_order face_recognition/FRClientGoal -- 4 "none"                  //exit

