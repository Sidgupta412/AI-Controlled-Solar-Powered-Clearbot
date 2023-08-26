# AI-Controlled-Solar-Powered-Clearbot

# Code for Image Processing and Object Detection using Rasberry PI-4

import cv2 as cv
import serial
bottle_cascade = cv.CascadeClassifier('cascade_HAAR_0-1.xml')
print(bottle_cascade)
low_battery = 0
cap = cv.VideoCapture(0)
com = 'COM4'
arduino = serial.Serial(port=com, baudrate=115200, timeout=10)
print(cap.get(cv.CAP_PROP_FRAME_HEIGHT))
print(cap.get(cv.CAP_PROP_FRAME_WIDTH))
if cap.isOpened():
print('camera found')
else:
print('camera not found')
# initial message
print('sending...')
arduino.write('0'.encode('utf-8'))
print(arduino.readline().decode('utf-8').rstrip())
# perform detection
def detection(f):
gray = cv.cvtColor(f, cv.COLOR_BGR2GRAY)
bottle = bottle_cascade.detectMultiScale(gray, 1.10, 4)
return bottle
# print location and send command
def location_in_frame(pos):
if pos == 'l':
arduino.write('1'.encode('utf-8'))
elif pos == 'r':
arduino.write('3'.encode('utf-8'))
elif pos == 'm':
arduino.write('2'.encode('utf-8'))
elif pos == '0':
arduino.write('0'.encode('utf-8'))
msg = arduino.readline().decode('utf').rstrip()
if msg == 'low':
print(msg)
return 1
else:
print(msg)
return 0
# find location in frame and return it
def get_position(orientation, p):
l = 0
m = 0
r = 0
if orientation == 'upright':
for (x, y, width, height) in p:
centre = x + width / 2
if 0 < centre <= 640 / 3:
l = l + 1
elif 640 / 3 < centre <= 2 * 640 / 3:
m = m + 1
elif 2 * 640 / 3 < centre <= 640:
r = r + 1
elif orientation == 'clockwise':
for (x, y, width, height) in p:
centre = y + height / 2
if 0 < centre <= 160:
l = l + 1
elif 160 < centre <= 320:
m = m + 1
elif 320 < centre <= 480:
r = r + 1
elif orientation == 'anticlockwise':
for (x, y, width, height) in p:
centre = y + height / 2
if 0 < centre <= 160:
r = r + 1
elif 160 < centre <= 320:
m = m + 1
elif 320 < centre <= 480:
l = l + 1
elif orientation == 'inverted':
for (x, y, width, height) in p:
centre = x + width / 2
if 0 < centre <= 640 / 3:
r = r + 1
elif 640 / 3 < centre <= 2 * 640 / 3:
m = m + 1
elif 2 * 640 / 3 < centre <= 640:
l = l + 1
else:
print('error')
if l == 0 and r == 0 and m == 0:
return '0'
elif l > 0 and r == 0 and m == 0:
return 'l'
elif l == 0 and r > 0 and m == 0:
return 'r'
elif l == 0 and r == 0 and m > 0:
return 'm'
elif l > 0 and r > 0 and m == 0:
if l > r:
return 'l'
elif r > l:
return 'r'
elif l == r:
return 'l'
elif l > 0 and r == 0 and m > 0:
if l > m:
return 'l'
elif m > l:
return 'm'
elif l == m:
return 'm'
elif l == 0 and r > 0 and m > 0:
if r > m:
return 'r'
elif m > r:
return 'm'
elif m == r:
return 'm'
elif l > 0 and r > 0 and m > 0:
if l > r and l > m:
return 'l'
elif r > l and r > m:
return 'r'
elif m > r and m > l:
return 'm'
elif l == r == m:
return 'm'

# main program
while cap.isOpened():
if low_battery == 1:
break
camera, frame = cap.read()
frame = cv.rotate(frame, cv.ROTATE_180)
# clockwise
clockwise_frame = cv.rotate(frame, cv.ROTATE_90_CLOCKWISE)
location_clockwise = detection(clockwise_frame)
position = get_position('clockwise', location_clockwise)
if position != '0':
low_battery = location_in_frame(position)
continue
# anticlockwise
anticlockwise_frame = cv.rotate(frame, cv.ROTATE_90_COUNTERCLOCKWISE)
location_anticlockwise = detection(anticlockwise_frame)
position = get_position('anticlockwise', location_anticlockwise)
if position != '0':
low_battery = location_in_frame(position)
continue
# inverted
inverted_frame = cv.rotate(frame, cv.ROTATE_180)
location_inverted = detection(inverted_frame)
position = get_position('inverted', location_inverted)
if position != '0':
low_battery = location_in_frame(position)
continue
# upright
location_upright = detection(frame)
position = get_position('upright', location_upright)
if position != '0':
low_battery = location_in_frame(position)
continue
# if object not found
low_battery = location_in_frame(position)
if low_battery == 1:
print('low battery, stopping program')
cap.release()

