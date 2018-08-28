#!/usr/bin/env python
import serial, binascii, time, json, requests

serverURL = "http://avior.globalinovasi.com/rfid/api/"

a = b"\xb0\x0c\x00\x00"
k1=b"\x1B\x2B\x00\x00\x00\x00\x00"
k2=b"\x1B\x32\x00\x00\x00\x00\x00"
#read 43 byte
k3=b"\x1B\x31\x00\x00\x00\x00\x00"
#baca 23 byte
Rn = b"\x1B\x2A\x00\x00\x00\x00\x00"
R1a = b"\x1B\x32\x00\x00\x00\x00\x00"
R1b = b"\x1B\x25\x00\x00\x00\x00\x00"

habis = b"\x1B\x25\x00\x00\x01\xF3\xF3"

arTag = []

###
# baca sekali:
# write R1a, baca 43 byte
# write R1b, baca 32 byte, baca 23 byte
# selama 8 byte pertama != b"\x1B\x25\x00\x00\x01\xF3\xF3" ada tag
# id tag byte ke-7 s/d 18

SRn = b""


ser = serial.Serial()
ser.baudrate = 115200
#ser.port = '/dev/cu.usbserial'
ser.port = 'COM4'
ser.timeout = 1
ser.open()

res = ser.write(k1)
#print(res)
res = ser.write(k2)
#print(res)
ser.read(43)
#print(hasil)
res = ser.write(k3)
ser.read(23)
ser.write(R1a)
ser.read(43)
ser.write(R1b)
time.sleep(5)
ser.read(32)

ada = True

print("Baca Tag")

jmlByte = ser.inWaiting()
if(jmlByte>0):
    if(jmlByte>8):
        while jmlByte>=23:
            kop = ser.read(6)
            tag = ser.read(12)
            sis = ser.read(5)
            stag = binascii.hexlify(tag)
            if stag not in arTag:
                arTag.append(stag)
                #print(stag)
            jmlByte = ser.inWaiting()
    else:
        ser.read(jmlByte);

sdata = json.dumps(arTag);
print(sdata)

#kirim data ke server
sdata = "{'tag': %s}" % (sdata)
req = requests.post(serverURL,data=sdata)
print(req.status_code, req.reason)

print(req.text)

print("SELESAI")
ser.close()
