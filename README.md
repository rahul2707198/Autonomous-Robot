import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from keras.models import Sequential
from keras.layers import Conv2D
from keras.layers import MaxPooling2D
from keras.layers import Flatten
from keras.layers import Dense
from keras.preprocessing.image import ImageDataGenerator
from keras.preprocessing import image
classifier=Sequential()
classifier.add(Conv2D(32,(3,3),input_shape=(64,64,3),activation='relu'))
classifier.add(MaxPooling2D(pool_size=(2,2)))
classifier.add(Flatten())
classifier.add(Dense(output_dim=128,activation='relu'))
classifier.add(Dense(output_dim=5,activation='sigmoid'))
classifier.compile(optimizer='adam',loss='binary_crossentropy',metrics=['accuracy'])
train_datagen=ImageDataGenerator(rescale=1./255,shear_range=0.2,zoom_range=0.2,horizontal_flip=True)
test_datagen=ImageDataGenerator(rescale=1./255)
training_set=train_datagen.flow_from_directory('C:/Users/pc/Downloads/datasets/training',target_size=(64,64),batch_size=32,class_mode='categorical')
test_set=test_datagen.flow_from_directory('C:/Users/pc/Downloads/datasets/test',target_size=(64,64),batch_size=32,class_mode='categorical')
classifier.fit_generator(training_set,steps_per_epoch=260,epochs=10,validation_data=test_set,validation_steps=260)


#2.  Green, Yellow, Red, STOP  signs and Obstacles prediction function from Above Built CNN model.
def predict(test_image):
test_image=image.img_to_array(test_image)
test_image=np.expand_dims(test_image,axis=0)
    result=classifier.predict(test_image)
training_set.class_indices
    if result[0][0]==1: # command for Green light 
        command="GO"
elif result[0][1]==1: # command for Yellow light
        command="WAIT"
elif result[0][2]==1: # command for Red Light
        command="STOP"
elif result[0][3]==1:                 # command for STOP sign
        command="STOP"
else:
     command="Obstacle"
return command

#3. Code Implementation for Lane detection through Hough Transform
 
import math
import cv2
import numpy as np 
import matplotlib.pyplot as plt

def display_lines(image,lines):
line_image=np.zeros_like(image)
    if lines is not None:
        for line in lines:
            x1,y1,x2,y2=line.reshape(4)
            cv2.line(line_image,(x1,y1),(x2,y2),(255,0,0),10)
    return line_image

def make_coordinates(image,line_parameters):
slope,intercept=line_parameteres
    y1=image.shape[0]
    y2=int(y1*(3/5))
    x1=int((y1-intercept)/slope)
    x2=int((y2-intercept)/slope)
    return np.array([x1,y1,x2,y2])

def average_slope_intercept(image,lines):
left_fit=[]
right_fit=[]
    for line in lines:
        x1,y1,x2,y2=line.reshape(4)
        parameters=np.polyfit((x1,x2),(y1,y2),1)
        slope=parameters[0]
        intercept=parameters[1]
        if slope<0:
left_fit.append((slope,intercept))
        else:
right_fit.append((slope,intercept))
left_fit_avg=np.average(left_fit,axis=0)
right_fit_avg=np.average(right_fit,axis=0)
left_line=make_coordinates(image,left_fit_avg)
right_line=make_coordinates(image,right_fit_avg)
    return np.array([left_line,right_line])

def region_of_interest(canny_img):
    height=canny_img.shape[0]
    polygons=np.array([
        [(0,height),(canny_img.shape[1],height),(canny_img.shape[1]//2,250)]
    ])
    mask=np.zeroes_like(canny_img)
    cv2.fillPoly(mask,polygons,255)

    ## Bitwise & b/w mask and canny image,to show only region of interest traced
     # by the polygon contour of the mask
masked_img=cv2.bitwise_and(canny_img,mask)
    return masked_img


def lane_detection(img):
    # Color image to GrayScale image conversion
gray_img = cv2.cvtColor(img, cv2.COLOR_RGB2GRAY) 

    ## Gaussian Blur
    # Higher the kernel, the more blur the outcome image will be.
kernel_size = 5
gauss_img = cv2.GaussianBlur(gray_img,(kernel_size, kernel_size), 0)

    ## Canny Edge Detection : It is used to detect boundaries of an image, through the gredientsof the image
low_threshold, high_threshold = [200, 300]
canny_img = cv2.Canny(gauss_img, low_threshold, high_threshold)
cropped_img=region_of_interest(canny_img)
    ## Hough Transform: to connect the dots of images by transporting the images to the Parameter Space
     # We have taken polar coordinates(rho,theta),in which we searched for intersecting lines.
    lines = cv2.HoughLinesP(cropped_img, rho=1, theta=math.pi/180,
                        threshold=15, np.array([]),        
minLineLength=30,
maxLineGap=40)

averaged_lines=average_slope_intercept(img,lines)

line_image=display_lines(img,averaged_lines)
combo_image=cv2.addWeighted(img,0.8,line_image,1,1)
    return combo_image

#4. Code  To Control Motors  : Robot  follows the command   Forward , Left_Turn , Right_Turn  and STOP.
import RPi.GPIO as gpio
import time
def init():
    gpio.setmode(gpio.BOARD)
    gpio,setup(7,gpio.OUT)
    gpio,setup(11,gpio.OUT)
    gpio,setup(13,gpio.OUT)
    gpio,setup(15,gpio.OUT)
def stop(tf):
    init()
    gpio.output(7,False)
    gpio.output(11,False)
    gpio.output(13,False)
    gpio.output(15,False)
    time.sleep(tf)
    gpio.cleanup()
    
def  forward(tf):   # Move Forward
    init()
    gpio.output(7,False)
    gpio.output(11,True)
    gpio.output(13,True)
    gpio.output(15,False)
    time.sleep(tf)
    gpio.cleanup()
def  reverse(tf):   # Move Reverse
  def  reverse(tf):   # Reverse back
    init()
    gpio.output(7,True)
    gpio.output(11,False)
    gpio.output(13,False)
    gpio.output(15,True)
    time.sleep(tf)
    gpio.cleanup()
def  turn_left(tf):   # Turn Left
    init()
    gpio.output(7,True)
    gpio.output(11,True)
    gpio.output(13,True)
    gpio.output(15,False)
    time.sleep(tf)
    gpio.cleanup()
def turn_right(tf):  # Turn Right
    init()
    gpio.output(7,False)
    gpio.output(11,False)
    gpio.output(13,False)
    gpio.output(15,True)
    time.sleep(tf)
    gpio.cleanup()
def  pivot_left(tf):   # Pivot  Left
    init()
    gpio.output(7,True)
    gpio.output(11,False)
    gpio.output(13,True)
    gpio.output(15,False)
    time.sleep(tf)
    gpio.cleanup()
def  Pivot_right(tf):  #  Pivot Right
    init()
    gpio.output(7,False)
    gpio.output(11,True)
    gpio.output(13,False)
    gpio.output(15,True)
    time.sleep(tf)
    gpio.cleanup()



    
#5. Driver Function  Of Lane detection ,Traffic Signals Detection And Give the Command to System

## Driver Function 

if __name__=="__main__": 
    command="RUN"
    cap=cv2.VideoCapture(0)
    left=0
    right=0
    while True:
ret,frame=cap.read()
lane_detected_img=lane_detection(frame)
        if lane_detected_img is not None:
            left=0
            right=0
            if predict(frame)=="GO": ## Green sign detected
                forward(100)  # move forward Function Call 
           elif predict(frame)=="WAIT":
                command="WAIT" 
           elif  predict(frame)== "STOP"            # RED Sign or STOP sign detected
                stop(1)
          else:	   # if any Obstacles found 
                  reverse(1)
                  pivot _left(1)
                  forward(1)
                 pivot_right(1)

        else:
            # Now turn left and check whether lane is present or not 
            if left==0:
                turn_left(1)    # 1 is the time frame  for left turn  function call
                left+=1
                continue
            else:
                if right==0:
                    turn_right(1)  # Right Turn 
                    right+=1
                else:
                    stop(1)       #stop function Call 
        if cv2.waitKey(1)==13:
            break

    #relaese camera and close windows
cap.release()
    cv2.destroyAllWindows() 





 
 
  
  
  
  
  
  
  
  
  
  
  
  
 
 
 

 



 
 
  
  
  
  
  
  
  
  
  
  
  
  
 
 
 

 

 
 

 
 

 
 

 
 
