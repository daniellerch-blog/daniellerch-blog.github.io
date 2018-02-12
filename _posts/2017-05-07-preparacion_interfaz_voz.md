---
layout: post
title: Preparación de la interfaz de voz
comments: true
author: admin
author_url: #
show: true
---

Una de las funciones principales de una casa inteligente es la capacidad de hablar,  tanto para entender lo que le decimos como para responder a nuestras peticiones. En este artículo veremos como establecer la configuración básica del entorno de voz. Para ello usaremos el lenguaje de programación Python y supondremos que disponemos de un sistema operativo basado en Debian.

Empezaremos con la **síntesis de voz**. He probado diferentes herramientas disponibles para GNU/Linux, como pueden ser [Festival](http://www.cstr.ed.ac.uk/projects/festival/) o [eSpeak](http://espeak.sourceforge.net/). Sin embargo, la voz en castellano que más me ha gustado es la de Pico TTS. 

Instalaremos:

```shell

$ sudo apt-get install libttspico-utils alsa-utils

```

Con lo que ya podemos generar un audió a partir de texto y reproducirlo con:

```shell

$ pico2wave -l es-ES -w test.wav "Buenos días, ¿que tal estás?" 
$ aplay test.wav

```

El **reconocimiento de voz** es un poco más complicado. Actualmente existen servicios de reconocimiento de voz de gran calidad, como los que ofrece [Google](https://cloud.google.com/speech/) o [Amazon](https://developer.amazon.com/alexa-voice-service). Pero en la medida de lo posible, me gustaría que este proyecto estuviese basado en software libre. Por lo tanto, aunque quizás en el futuro añada soporte para servicios de voz de terceros, partiremos del software [PocketSphinx](https://github.com/cmusphinx/pocketsphinx), de la [Carnegie Mellon University](http://www.cmu.edu/). Este softare y el modelo de voz en castellano no tiene la calidad de los servicios de Amazon y Google, pero nos sirve para empezar.

Empezaremos instalando los paquetes necesarios:

```shell

$ sudo apt-get install portaudio19-dev libpulse-dev
$ sudo pip install pyaudio
$ sudo pip install pocketsphinx

```


Veamos como podemos usar el reconociminto de voz. A continuación, pongo un pequeño script que configura PocketSphinx, abre el archivo de audio "test.wav" que hemos generado en el paso anterior e intenta traducir la voz a audio. 

Para que funcione correctamente, debemos tener en el mismo directorio el modelo de voz en castellano, que se puede descargar desde [aquí](ttps://sourceforge.net/projects/cmusphinx/files/Acoustic%20and%20Language%20Models/Spanish/).



```python                                                                              

from pocketsphinx.pocketsphinx import Decoder                                    
                                                                                 
config = Decoder.default_config()                                                
config.set_string('-hmm', 'cmusphinx-es-5.2/model_parameters/voxforge_es_sphinx.cd_ptm_4000')
config.set_string('-lm', 'es-20k.lm.gz')                                         
config.set_string('-dict', 'es.dict')                                            
config.set_string('-logfn', '/dev/null')                                         
decoder = Decoder(config)                                                        
                                                                                 
# Decode streaming data.                                                         
decoder = Decoder(config)                                                        
decoder.start_utt()                                                              
stream = open('test.wav', 'rb')                                               
while True:                                                                      
  buf = stream.read()                                                            
  if buf:                                                                        
    print "Decode!"                                                              
    decoder.process_raw(buf, False, False)                                       
  else:                                                                          
    break                                                                        
decoder.end_utt()                                                                
                                                                                 
words=[seg.word for seg in decoder.seg()]                                        
                                                                                 
string=''                                                                        
for w in words:                                                                  
    if w in ["<s>", "</s>"]:                                                     
        continue                                                                 
    if w in ["<sil>"]:                                                           
        continue                                                                 
    string+=w+' '                                                                
                                                                                 
print string                                                                     
             
```

Si ejecutamos el script obtenemos algo como esto:

```shell

$ python voice_recognition.py
buenos días qué tal estás 

```


Si queremos grabar un audio con nuestra voz, en lugar de usar la generada por PicoTTS, podemos hacerlo con el siguiente comando:

```shell

$ arecord test.wav

```

Con todo esto funcionando ya tenemos lo básico para empezar a trabajar con la voz. Sin embargo, puede que todo esto no funcione a la primera. Dependiendo de la distribución Linux que tengamos y, principalmente, del hardware, nos podemos encontrar con diferentes problemas. Resolverlos, queda como tarea para el lector.











