#coding=utf-8
import RPi.GPIO as GPIO
import socket
import time
import threading
import os

#Motor drive interface definition
#you can change it to your style
ENA = 13	#//L298 ENA
ENB = 12	#//L298 ENB

IN1 = 19	#//Motor1 out1
IN2 = 16	#//Motor1 out2
IN3 = 21	#//Motor2 out1
IN4 = 26	#//Motor2 out2

nowLeftSpeed = float(0)
nowRightSpeed = float(0)
runTime = float(0.7)
isFinished = False
isClosed = False

#change the port as you want
adr_p = ('0.0.0.0', 1234)
count = 5
size = 1024
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(adr_p)
s.listen(count)
s.settimeout(30)
con,addr = s.accept()
con.settimeout(30)

#Specify encoding rules
GPIO.setmode(GPIO.BCM) 

#Pin initial setting
def setup():
    GPIO.setmode(GPIO.BCM)
    global pwm_L
    global pwm_R
    
    #Disable warnings when the monitoring code is non default
    GPIO.setwarnings(False) 
    
    #Set pin mode to output
    GPIO.setup(ENA, GPIO.OUT)
    GPIO.setup(IN1, GPIO.OUT)
    GPIO.setup(IN2, GPIO.OUT)
    
    GPIO.setup(ENB, GPIO.OUT)
    GPIO.setup(IN3, GPIO.OUT)
    GPIO.setup(IN4, GPIO.OUT)
    
    try:
        pwm_L = GPIO.PWM(ENA, 2000)
        pwm_R = GPIO.PWM(ENB, 2000)
        pwm_L.start(0)
        pwm_R.start(0)
    except Exception as e:
        print('PWM SET ERROR:')
        print(e)


def runL(speed):
    if(speed == 0 or speed == -0):
        pwm_L.ChangeDutyCycle(0)
    else:
        if(speed < 0):
            GPIO.output(IN1, 0)
            GPIO.output(IN2, 1)
            pwm_L.ChangeDutyCycle(-speed)
        else:
            GPIO.output(IN1, 1)
            GPIO.output(IN2, 0)
            pwm_L.ChangeDutyCycle(speed)

def runR(speed):
    if(speed == 0 or speed == -0):
        pwm_R.ChangeDutyCycle(0)
    else:
        if(speed < 0):
            GPIO.output(IN3, 1)
            GPIO.output(IN4, 0)
            pwm_R.ChangeDutyCycle(-speed)
        else:
            GPIO.output(IN3, 0)
            GPIO.output(IN4, 1)
            pwm_R.ChangeDutyCycle(speed)


class leftEngineThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        while 1:
            if(isFinished):
                break
            print('LeftEngine Run At: ', nowLeftSpeed)
            runL(nowLeftSpeed)
            time.sleep(runTime)
            
class rightEngineThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        while 1:
            if(isFinished):
                break
            print('RightEngine Run At: ', nowRightSpeed)
            runR(nowRightSpeed)
            time.sleep(runTime)


class recThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        global nowLeftSpeed
        global nowRightSpeed
        global isClosed
        speedData = []
        while 1:
            try:
                msg = con.recv(size)
                if msg == '' or msg == '   ' or msg == '  ' or msg.isspace():
                    print('RECEIVER ERROR')
                    isClosed = True
                    break
                msg = msg.decode('utf-8')
                #print('msg: ', msg)
                speedData.clear()
                for i in msg.split(' '):
                    if i != "":
                        speedData.append(int(i))
                nowLeftSpeed = speedData[0]
                nowRightSpeed = speedData[1]
            except Exception as e:
                print('RECEIVER ERROR:')
                print(e)
                isClosed = True
                break


class sendThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        global isFinished
        global isClosed
        while 1:
            print('sendThread')
            if isClosed:
                break
            try:
                text = input('send text >> :')
                if text == 'q'or text == 'Q'or text == 'w'or text == 'W':
                    print('SENDER QUIT')
                    isFinished = True
                    break
                con.sendall(bytes(text+'\n' ,'utf-8'))
            except Exception as e:
                isClosed = True
                print('SENDER ERROR')
                break

class mjpg (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        output = os.popen('mjpg_streamer -i "./input_uvc.so -r 1280x768 -f 25 -n -y" -o "./output_http.so -p 8080 -w /usr/local/www"')
        print(output.read())


def main():
    global con
    global isFinished
    global isClosed
    print("CONNECTION: ",con)
    setup()
    send = sendThread(1, "sendThread-1", 1)
    recv = recThread(2, "recThread", 2)
    leftEngine = leftEngineThread(3,"LeftEngine",3)
    rightEngine = rightEngineThread(4,"RightEngine",4)
    leftEngine.start()
    rightEngine.start()
    mp = mjpg(5, "mjpg", 5)
    mp.start()
    send.start()
    recv.start()
    while 1:
        time.sleep(0.75)
        if isFinished:
            print('connection closed 222')
            con.shutdown(2)
            con.close()
            s.shutdown(2)
            s.close()
            time.sleep(1)
            break

        if isClosed:
            print('connection closed 111')
            con,addr = s.accept()
            time.sleep(runTime)
            send.join()
            recv.join()
            send = sendThread(1, "Thread-1", 1)
            recv = recThread(2, "Thread-2", 2)
            send.start()
            recv.start()
            isClosed = False
            print('connection rebuilt')
    GPIO.cleanup()



if __name__ == '__main__':
    main()
