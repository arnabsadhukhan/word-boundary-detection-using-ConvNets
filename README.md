# word-boundary-detection-using-ConvNets
WORD BOUNDARY DETECTION

WE WANT TO CREATE A MODEL WHICH CAN TAKE THE AUDIO SIGNAL AND CAN TELL THAT WHERE ARE THE WORDS STARTING AND ENDING POSITION
FOR DOING THAT LETS LOOK AT A SIGNAL

![image 1](Images/1.png?raw=true "Title")

AS WE CAN SEE WE CAN`T GET ANY CLUE WHERE THE WORD BREAKS WILL BE BY JUST LOOKING THE RAW AUDIO SIGNAL
ANOTHER PROBLEM IS DIFFERENT AUDIO SIGNALS HAVE DIFFERENT LENGTHS SO WE HAVE TO FIND A SOLUTION FOR THAT
THIS AUDIO SIGNAL HAVE 5 WORDS WE CAN SEE THAT FROM THE FIGURE BELOW
 
![image 2](Images/2.png?raw=true "Title") 
 
WE HAVE 150 AUDIO SIGNALS LIKE THAT AND WE MANUALLY TAKE THE EACH WORDS START TIME END ITS DURATION AND SAVE IT AS A CSV FILE 
THEN WE CREATE A PROFILE OF WORD FROM THE CSV FILES 

THE CSV FILE LOOK LIKE THIS
Name	Start	Duration	Time Format	
Marker 01	0:00.202	0:00.491		
Marker 02	0:00.718	0:00.517	
Marker 03	0:01.537	0:00.532	
Marker 04	0:02.196	0:00.464		
Marker 05	0:02.660	0:00.381	 

ALGO:
Length = length ( audio  signal )
Word profile = zeros ( shape = ( Length ))
For i IN (NO OF MARKERS):
	END TIME = START_TIME +  DURATION
	WORD_PROFILE [ START_TIME   :   END TIME ]  =   1
END

IN THIS WAY WE CREATE A WORD PROFILE OF THE SIGNAL WHICH LOOK LIKE THIS
 
![image 3](Images/3.png?raw=true "Title") 
 
NOTE: THE WORD PROFILE HAVE SAME LENGTH AS THE AUDIO SIGNAL.

NOW THE THE WORD PROFILE HOLDS ALL THE INFORMATION 
•	WHERE THE WORDS ARE 
•	HOW MANY WORDS ARE THERE
•	EVERY WORDS STARTING AND ENDING TIME OR POSITION


SO NOW THE MAIN IDEA IS 
IF WE GIVE THE SIGNAL TO A MODEL AND WHICH OUTPUTS WORD PROFILE. 
MEANS THE SIHNAL IS THE INPUTS
AND WORD PROFILE IS THE OUTPUT
  SIGNAL                                                             MODEL				    WORD PROFILE
   
![image 4](Images/4.png?raw=true "Title")



BUT ITS NOT EASY BECAUSE
LET SAY A NORMAL AUDIO LENGTH IS 3 Sec
AND IF ITS SAMPLING FREQUENCY IS  22050/SEC
SO WE HAVE A LENGTH OF AUDIO SIGNAL OF   3  X   22050     =  66150
AND THE LENGTH OF WORD PROFILE IS ALSO SAME 66150


FEEDING RAW AUDIO SIGNAL IS NOT A GOOD IDEA TO TRAIN A MODEL SO WE HAVE TO DO SOMETHING ELSE..

CREATE THE TRAING DATA FOR THE MODEL OR PRE-PROCESSING OF AUDIO 
LETS TAKE A SIGNAL
 
![image 5](Images/1.png?raw=true "Title")

![image 6](Images/5.png?raw=true "Title")

                              
AND DIVIDE THE SIGNAL IN 0.5 SEC CHUNKS
NOW DO THE SAME THING TO WORD PROFILE

![image 7](Images/3.png?raw=true "Title")
 
 ![image 8](Images/6.png?raw=true "Title")
 
                                       
SO NOW  THIS IS OUR INPUTS STEP BY STEP
    FIRST STEP	SECOND STEP     THIRD		FORTH ………………  SO ON………
                              
![image 9](Images/7.png?raw=true "Title")                                                         
                                       
THIS IS OUR MODEL OUTPUTS

NOW WE HAVE 0.5 SEC INPUTS AND 0.5 SEC OUTPUT 
IF SAMPLING FREQUENCY    = 22050/SEC
MEANS INPUT SHAPE = 22050 * 0 . 5  = 11025 SAMPLES
AND OUTPUT (WORD PROFILE) SHAPE = 11025 SAMPLES

THE INPUT SHAPE AND OUTPUT IS REDUCED BUT STILL THE OUTPUT SHAPE IS QUITE BIG FOR A MODEL LETS REDUCED IT MORE 



LETS LOOK AT ONE 0.5 SEC WORD PROFILE

![image 10](Images/8.png?raw=true "Title")
 
THE WHOLE SQUENCE IS 11025 SAMPLES LONG
DID WE REALLY NEED THIS MUCH SAMPLES TO MAKE THIS PROFILE THE ANSWER IS NO 

![image 11](Images/9.png?raw=true "Title")
 
THE RED DOT ARE THE 5 ms INTERVAL POINTS
THE SEQUENCE IS 500 ms SO IF WE TAKE ONE VALUE FOR EVERY 5 ms INTERVAL WHICH WILL BE 0 OR 1
THIS WILL BE ENOUGH VALUES TO REPRESENT THE PROFILE
NOW LOOK AT THE WORD PROFILE WHICH VALUES ARE ONLY TAKEN AT 5ms INTERVAL

![image 12](Images/10.png?raw=true "Title")
 

THOSE POINTS ARE ENOUGH TO REPRESENT THE SIGNAL AND THE TWO SIGNAL ARE ALMOST SAME
NOW SEE WE HAVE WORD PROFILE OF 0.5 SEC OR 500 ms WHICH LENGTH IS 11025
NOW WE TAKE ONLY THE 5ms VALUE SO NO OF POINTS WILL BE   500 ms /5 ms = 100 SAMPLES 


11025 TO 100 POINTS THIS IS SIGNIFICANT CHANGE IN OUTPUT 
ONE OF THE ADVANTAGE IS OF LESS NO OF OUTPUT IS THE MODEL WILL BE MORE ACURATE FOR SMALL NO OR TRAINING DATA 


NOW WE HAVE 11025 NO OF POINTS FOR INPUT SIGNAL FOR MODEL
AND 100 NO OF OUTPUT POINTS 

NOW WE CAN TRAIN THE MODEL BUT WE HAVE ONE PROBLEM LEFT , TRAING THE MODEL WITH RAW AUDIO IS NOT A GOOD IDEA WE HAVE TO TRANSFORM THE AUDIO FROM TIME DOMAIN TO ANY OTHER 
THE TRANSFORMATION OPTIONS ARE 
•	FFT
•	STFT
•	SPECTROGRAM
•	MEL SPECTROGRAM
•	MFCC
AMONG THESE MEL SPECTROGRAM GIVES THE BEST RESULT SO WE PERFORM THIS TRANSFORMATION

TO GET THE  0.5 SEC MEL SPECTROGRAM WE NEED TO SET TWO PARAMETERS
librosa.feature.melspectrogram(y=audio_samples_array, sr=fs, n_mels=256,fmax=8000,hop_length=int(fs*window),n_fft=1024)

THIS GIVE 256x100 MATRIX FOR EVERY 0.5 SEC





 SO WE HAVE A [256x100] MATRIX      NOW THIS MATRIX IS THE INPUT TO THE MODEL 

    FIRST STEP	SECOND STEP     THIRD		FORTH ………………  SO ON………
![image 13](Images/5.png?raw=true "Title")                             
Mel Spectrogram [256x100] MATRIX
![image 14](Images/11.png?raw=true "Title")                                                                            
FEEDING THE MEL SPECTROGRAM MATRICES TO MODEL ONE BY ONE
![image 15](Images/12.png?raw=true "Title")                                                        
REDUCED 100 POINT  WORD PROFILE
![image 16](Images/6.png?raw=true "Title")                                        




MODEL ARCHITECTURE:



model = Sequential()

model.add(Conv2D(64,(3,3),padding='same',input_shape=(256,100,1)))
model.add(BatchNormalization())
model.add(LeakyReLU())
model.add(Dropout(0.25))

model.add(Conv2D(32,(3,3),padding='same'))
model.add(BatchNormalization())
model.add(LeakyReLU())
model.add(Dropout(0.25))

model.add(Conv2D(16,(3,3),padding='same'))
model.add(BatchNormalization())
model.add(LeakyReLU())
model.add(Dropout(0.25))

model.add(Conv2D(8,(3,3),padding='same'))
model.add(BatchNormalization())
model.add(LeakyReLU())
model.add(Dropout(0.25))

model.add(Conv2D(4,(3,3),padding='same'))
model.add(BatchNormalization())
model.add(LeakyReLU())
model.add(Dropout(0.25))

model.add(Flatten())

model.add(Dense(512,activation='relu'))
model.add(Dropout(0.25))
model.add(Dense(256,activation='relu'))
model.add(Dense(100,activation='sigmoid'))





AFTER TRAIN THE MODEL WE GET 83 % VALIDATION ACCURACY
NOW TIME TO TEST THE MODEL
MODEL MODEL IS CABAPLE OF ONLY CREATE  THE WORD PROFILE OF 0.5 SEC SO WE PRECT ALL THE SEQUEENCE OF WORD PROFILES AND COMBINE AT THE END 
THIS PROFILE IS CREATE BT MODEL 
![image 17](Images/13.png?raw=true "Title") 
 
HERE ARE SOME MORE RESULTS:

![image 18](Images/14.png?raw=true "Title") 
![image 19](Images/15.png?raw=true "Title") 

 

