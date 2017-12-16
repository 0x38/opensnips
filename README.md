# OpenSnips

## Overview

!! This project is a work in progress. The documentation is ahead of the function ;)

This repository is a collection of projects integrating [Snips AI](http://snips.ai)  and [RASA AI](http://rasa.ai) and [Kaldi](https://github.com/alumae/kaldi-gstreamer-server) and [Meeka@home](http://meekamusic.com) to create a 100% open source implementation of the [Hermes MQTT protocol](https://github.com/snipsco/snips-platform-documentation/wiki/6.--Miscellaneous#hermes-protocol) used by Snips. 

<br>

The repository developed to allow running the Hermes protocol on a Linux laptop or server. Most of the services will work fine on a Raspberry Pi but some hacks(temporary swap file) are required for building some services.

By using RASA core as the basis of a replacement for the Snips Skills server, it is possible to use machine learning to control the flow of conversation and develop extended interactions that go beyond confirmations and form wizards.

By using RASA NLU and Kaldi ASR, it is possible to build training pipelines locally so that changing ASR and/or NLU models is as easy as saving a file. Kaldi also provides a distinct websockets interface that can be used from Android with [Konele](http://kaljurand.github.io/K6nele/about/).

The provided hotword service is extended to use piwho to identify the speaker from their voice (ack [Greg](https://github.com/oziee/hotword) ).


<br><br>
Note that the official Snips is highly optimised to perform with large data sets on minimum hardware and includes quality software with error checking, logging and testing and is suitable for commercial deployments.

This repository is intended for developers who want to hack on Snips.

__Many thanks to the team at snipsco for their ongoing contribution to realising the future of talking to our tech. Namaste.__






<br>



__The repository includes: __

- Dockerfiles to build 
    - an multi architecture image based on Debian Jessie or Raspbian Stretch supporting the installation of Snips.
    - a multi architecture image supporting installation of RASA, Kaldi, Snowboy.
    - a multi architecture image for Pulseaudio.
    - a Kaldi image that incorporates the English language models.
    
- Python scripts implementing snips MQTT services (docker-images/rasa/snips_services/*)
    - audioserver
    - tts
    - hotword
    - asr (using Kaldi)
    - nlu (using RASA)
    - dialog
    - controller (using RASA core)
    
- docker-compose.yml file to start a suite including 
    - snips
    - replacement server for hotword detector using snowboy (snips_hotword_snowboy)
    - replacement server for NLU processor using rasa NLU (snips_rasa_nlu)
    - pulseaudio server to share the sound.
 

## Architecture/Platform Support

The Dockerfiles build on both arm7(Raspberry Pi) and x86_64(Linux desktop/server).

It is apparently possible to install pulseaudio on MS Windows and MacOSX which should allow the suite to be used with Docker on other platforms than Linux.


    
## Quick Start

To get started 

!! You will need at least 16G of storage space to install and run the suite.

### Docker
- [Install docker](https://www.google.com.au/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiPkYafmt3XAhUKJZQKHU3DBO4QFggoMAA&url=https%3A%2F%2Fdocs.docker.com%2Fengine%2Finstallation%2F&usg=AOvVaw3LbZ234MXDYJLII4P-TXAZ)



### To run
- ```pip install docker-compose```
- ```git clone https://github.com/syntithenai/opensnips.git```
- ```cd opensnips```
- ```docker-compose up```
- OR where pulseaudio is running on the host without extra config
- ```pasuspender -- docker-compose up```
- OR modify the environment variables and host mounts for the pulseaudio container in docker-compose.yml to use the host pulseaudio system.



### Build
- To build rasa on a raspberry pi you will need some swap memory. DO NOT LEAVE THIS ENABLED OR YOU WILL KILL YOUR SD CARD.

```

dd if=/dev/zero of=/swapfile bs=1M count=1024 # For 1GB swap file

mkswap /swapfile

swapon /swapfile

docker-compose build <service key>

```

__when finished be sure to      swapoff /swapfile; rm /swapfile__




    
## Snowboy
The docker-compose file contains environment variables to configure snowboy including 

- a path to model file. Create a model file at [https://snowboy.kitt.ai/](https://snowboy.kitt.ai/)
- site id for multiroom snips
- hotword ID 
```    environment:
        - HOTWORD_MODEL=/opt/snips_hotword_snowboy/resources/snowboy.umdl
        - SITE_ID=default
        - HOTWORD_ID=snowboy
```

## Snips

The generic asr model is built into the snips image. To override it, use docker-compose to mount a host volume containing a different model.

The image comes with a music player assistant as an example. Currently the snips assistant can be overridden by volume mount in docker-compose. In the future, the snips assistant files will be generated based on rasa stories.

Similarly the config file is a volume mount.

```
volumes:
            # generic model is built into image, override with other models here
            #- /home/stever/projects/snips-asr-model-en-500MB/snips-asr-model-en-500MB:/usr/share/snips/assistant/custom_asr
            # snips config
            - ./docker-compose/snips/config/assistant:/usr/share/snips/assistant
            - ./docker-compose/snips/snips.toml:/etc/snips.toml
```    

### Audio Server
snips audio server pushes 256 samples long frames, signed 16bit, mono, 16000Hz
each frame in its own .wav container




    
## RASA    

```       
       environment:
            - NLU_TRAINING_FILE=/opt/rasa/data/nlu-model/stories.md
            - NLU_CONFIG_FILE=/opt/rasa/data/nlu-model/config.json
            - NLU_MODEL_FOLDER=/opt/rasa/data/nlu-model/default/model_20171125-071720
 ```    


## Sound Configuration

In /etc/asound.conf, types dmix and dsnoop are fine for mixing/sharing device access across multiple services running natively but inside docker, the first container locks the sound device.

To allow multiple containers shared access to sound inside Docker a container running pulseaudio is included.

Other containers in the docker-compose suite can access the server through a shared socket. 

The server can be configured to use the sound hardware direct(default) or pulseaudio on the host system by editing the docker-compose file to remove comments and update to the IP address of the docker host and the path to the pulse cookie on the host.

```
 # proxy for host pulseaudio server
        #environment: ['PULSE_SERVER=192.168.1.100']
        volumes: 
            - ./pulse:/tmp/pulse
            # proxy for host pulseaudio server auth cookie
            #- /home/stever/.config/pulse/cookie:/root/.config/pulse/cookie
```

    
    
## Roadmap

My goal is to better understand what is possible in conversational UI by developing a user interface that
brings together RASA story telling format and snips skills.

My previous experience develop a voice first music player using dialogflow was very command like. For speech interactions to be widely accepted, recognition needs to be much more flexible and forgiving. I'm hopeful that RASA core will provide that flexibility.

As a starting point a minimal text format.

- Many stories serve as the training data of what actions to take based on what intents are triggered.
- Each story starts with ##
- An interaction starts with a * and the name of the intent.
- Example sentences preceded by = follow (used to generate NLU config)
- Actions sentences preceded by - by default return the text and where the text starts with an _ are executed (snips skills)


With the story format, confirmations, Yes/No responses, form wizard (slot filling) stories and more are possible.



### For example
```

## play some music
* play music
  = gimme some tunes
  = play some music
  - ok playing some random music
  - _play_music
  

## play some jazz music
* play music [genre=pop]
  = i want to hear some pop music
  = play some pop music
  - ok playing some pop music
  - _play_music

## play music by artist
* play music [artist=Josh Woodward]
  = i want to hear something by Josh Woodward
  = play some music by Josh Woodward
  - ok playing some music by Josh Woodward
  - _play_music

## clear the playlist
* clear the playlist
  - do you really want to clear the playlist?
* agree
  - ok clearing the playlist
  - _clearplaylist

```





# Links


## DeepSpeech
https://github.com/mozilla/DeepSpeech
https://hub.docker.com/r/voidspacexyz/deepspeech/builds/


### Kaldi
- https://github.com/alumae/kaldi-gstreamer-server
- https://hub.docker.com/r/jcsilva/docker-kaldi-gstreamer-server/
- or 
- https://github.com/hipstas/kaldi-pop-up-archive
- https://github.com/achernetsov/kaldi-docker-example
- https://github.com/alumae/gst-kaldi-nnet2-online

https://chrisearch.wordpress.com/2017/03/11/speech-recognition-using-kaldi-extending-and-using-the-aspire-model/


!!! one shot embedded model
https://github.com/jcsilva/docker-kaldi-gstreamer-server/issues/8

!!! aspire model
https://github.com/jcsilva/docker-kaldi-gstreamer-server/issues/15

#### Client

- http://kaljurand.github.io/K6nele/about/

- https://github.com/Kaljurand/dictate.js



## Alternative Suite
http://alex.readthedocs.io/en/master/index.html
https://github.com/UFAL-DSG

https://n0where.net/free-open-source-siri-project-sirius/
https://github.com/claritylab/lucida




## Reading
http://www.nltk.org/book/



## Corpus
http://www.openslr.org/12



    
### WIP    
- skills server using RASA core and story format described below to listen for hermes/nlu/intentParsed and take actions based on stories.
- web server with site for editing stories to generate rasa_nlu, rasa_core and snips stubs from a story.
