# WIRELESS-SOUND-CONTROLS
COINCENT PROJECT
#WIRELESS SOUND CONTROL PROJECT

#COINCENT-MAJOR PROJECT

#NAME: VISHAL KHUMAR P D

import cv2

import mediapipe as mp      #detects hands and detect the fingers on basis of landmarks.

import math

import numpy as np

from ctypes import cast, POINTER
from comtypes import CLSCTX_ALL
from pycaw.pycaw import AudioUtilities, IAudioEndpointVolume

devices = AudioUtilities.GetSpeakers()
interface = devices.Activate(
    IAudioEndpointVolume._iid_, CLSCTX_ALL, None)
volume = cast(interface, POINTER(IAudioEndpointVolume))

volume.GetMute()
volume.GetMasterVolumeLevel()
volume.GetVolumeRange()
vol=0
volbar=300
volPer=0



mpDraw=mp.solutions.drawing_utils

mpHands=mp.solutions.hands

hands=mpHands.Hands()

cap=cv2.VideoCapture(0)    #0 indicates the default camera in the system

while True:
    
    success,img=cap.read()

    imgRGB=cv2.cvtColor(img,cv2.COLOR_BGR2RGB)  

    results=hands.process(imgRGB)  #for processing the image

    a=results.multi_hand_landmarks

    #print(a)

    

    if a:

        for handLms in a:
            lmList=[]
            for id, lm in enumerate(handLms.landmark):
                
                #print(id, lm)
                h,w,c=img.shape     #h=height,w=width,c=channel
                
                #print(h,w,c)
                #print('\n')
                
                cx,cy=int(lm.x*w), int(lm.y*h)    #for getting the coordinates
                #print(id, cx, cy)
                
                lmList.append([id,cx,cy])
            #print(lmList)
                
            if lmList:
                x1,y1=lmList[4][1],lmList[4][2]
                x2,y2=lmList[8][1],lmList[8][2]
                cv2.circle(img, (x1,y1),10,(2,6,233),cv2.FILLED)
                cv2.circle(img, (x2,y2),10,(2,6,233),cv2.FILLED)
                cv2.line(img , (x1,y1), (x2,y2), (205,55,0), 4)

                length=math.hypot((x2-x1),(y2-y1))          #function for calculating distance between two coordinates
                #print(length)

                volRange=volume.GetVolumeRange()
                minVol= volRange[0]
                maxVol=volRange[1]

                vol=np.interp(length, [30,300], [minVol,maxVol])
                volbar=np.interp(length, [30,300], [300,130])
                volPer=np.interp(length, [30,300], [0,100])
                volume.SetMasterVolumeLevel(vol, None)

                if length<30:
                    cv2.putText(img,'MIN',(20,400),cv2.FONT_HERSHEY_COMPLEX,1,(0,255,0),3)               #For indication of minimum volume
                if length>300:
                    cv2.putText(img,'MAX',(30,60),cv2.FONT_HERSHEY_COMPLEX,1,(0,255,0),3)               #For indication of maximum volume

                    
            cv2.rectangle(img,(30,130),(85,300),(205,55,0),3)                  #creating volume bar on image screen
            cv2.rectangle(img,(30,int(volbar)),(85,300),(205,55,0),cv2.FILLED)
            if length<300:
                cv2.putText(img, f'{int(volPer)}%',(25,350),cv2.FONT_HERSHEY_COMPLEX,1,(2,6,233),3)    #for showing the volume percentage
            if length>300:
                cv2.putText(img, f'{int(volPer)}%',(25,110),cv2.FONT_HERSHEY_COMPLEX,1,(2,6,233),3)    #for showing the volume percentage
       
        

    
    
    cv2.imshow("Images",img)
    
    cv2.waitKey(1)               #helps in opening camera

#length range is from 30 to 300

#If length = 30, then volume should be zero ====> volRange= -65.25

#If length=300, then the volume must be maximum ====> volRange= 0.0
    
