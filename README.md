# gstreamer-rtp-streaming
simple working with rtp and gstreaming

for streaming audio:

    gst-launch-1.0 autoaudiosrc ! audioconvert ! rtpL24pay ! udpsink host=<RECEIVER_IP> port=<PORT>
    
streaming audio to multiple destinations:
    
    gst-launch-1.0 autoaudiosrc ! audioconvert ! rtpL24pay ! tee name=t ! queue ! udpsink host=<RECEIVER_IP_1> port=<PORT> \
        t. ! queue ! udpsink host=<RECEIVER_IP_2> port=<PORT> \
        t. ! queue ! udpsink host=<RECEIVER_IP_3> port=<PORT>
        
receiving audio:

    gst-launch-1.0 udpsrc port=<PORT> caps="application/x-rtp,channels=(int)2,format=(string)S16LE,media=(string)audio,payload=(int)96,clock-rate=(int)44100,encoding-name=(string)L24" ! \
    rtpL24depay ! audioconvert ! autoaudiosink sync=false
    
receiving rtp audio and stream it to another destination:
     
     gst-launch-1.0 udpsrc port=<PORT> caps="application/x-rtp,channels=(int)2,format=(string)S16LE,media=(string)audio,payload=(int)96,clock-rate=(int)44100,encoding-name=(string)L24" ! \
     rtpL24depay ! audioconvert ! rtpL24pay ! udpsink host=<RECEIVER_IP> port=<PORT>
     
receiving rtp audio and local Mic, mixing them and play them with speaker:

    gst-launch-1.0 autoaudiosrc ! audioconvert ! audiomixer name=mix ! autoaudiosink \
      udpsrc port=<PORT> caps="application/x-rtp,channels=(int)2,format=(string)S16LE,media=(string)audio,payload=(int)96,clock-rate=(int)44100,encoding-name=(string)L24" ! \
      queue ! rtpL24depay ! audioconvert ! mix.
      
receiving rtp audio and local Mic, mixing them and stream them with rtp:

    gst-launch-1.0 audiomixer name=mix ! audioconvert ! rtpL24pay ! udpsink host=<RECEIVER_IP> port=<PORT> \
        udpsrc port=<PORT> caps="application/x-rtp,channels=(int)2,format=(string)S16LE,media=(string)audio,payload=(int)96,clock-rate=(int)44100,encoding-name=(string)L24" ! \
        queue ! rtpL24depay ! audioconvert ! queue ! mix. autoaudiosrc ! audioconvert ! mix.
        
        
    
receving rtp audio and local Mic, mixing them and stream them to multiple destinations with rtp:

    gst-launch-1.0 audiomixer name=mix ! audioconvert ! rtpL24pay ! tee name=t ! queue autoaudiosrc ! audioconvert ! mix. t. ! queue ! \
    udpsink host=<DESTINATION_IP_1> port=5000 t. ! queue ! udpsink host=<DESTINATION_IP_2> port=5000 t. ! queue \
    udpsrc port=5001 caps="application/x-rtp,channels=(int)2,format=(string)S16LE,media=(string)audio,payload=(int)96,clock-rate=(int)44100,encoding-name=(string)L24" !\
    queue ! rtpL24depay ! audioconvert ! mix.
