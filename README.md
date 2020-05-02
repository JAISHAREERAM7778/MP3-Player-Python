# THE View.py FILE
The third module that we are going to code is the View module which will act allow the user to interact witht the application . It will contain buttons for adding song to the playlist , playing a song, removing a song from the playlist as well as buttons for setting/unsetting favourites . Also recall that these buttons will call respective methods of Player class to implement their functionality so we will require a Player object to be created in the View 

STEPS TO BE DONE BEFORE View.py
Before writing logic for View.py , we must create our own Exception class which will be used whenever the user takes an action like playing a song or removing a song etc without selecting the song.
To do this we require to create a module called MyException containing our own Exception class called NoSongSelectedException.
Following is it's code:
class NoSongSelectedError(Exception):
    pass


STEPS TO BE DONE IN View.py
The View module primarily has 3 tasks :
I.  Set Up the UI
II. Managing the user actions
III. Interacting with the Player class 
Moreover , we already have the UI designed , so we just have to do step II and step III
So the View module will do the following:
1. Rename the class Toplevel1 to View
2. This class will have the following responsibilities:
	A. Instantiate the Player object .
	B. Disable the addfavourite, removefavourite and loadfavourite Buttons if the database connection is not opened 	C. Set up the event handlers for all the Buttons and Listbox
	D. Provide method for handling closing of window so that we can get user confirmation before closing 
	E. Provide instance methods for handling song progress using MultiThreading
	
         
Based on the above discussion following will be the methods in our View class:

1. setup_player(): For instantiating the Player object , enabling or disabling the addfavourite, removefavourite and loadfavourite Buttons and setting up event handlers for all other Button

2. change_volume():For setting the volume level as selected by the user

3. add_song(): For adding song in the playlist.

4. show_song_details(): For displaying song title and song length in the UI.

5. play_song():For playing the currently selected  song
 

6. list_double_click():For playing the currently double clicked song

7. stop_song():For stopping the song currently being played.

8. pause_song():For pausing the song currently being played.

9. close_window(): For closing the app

10. remove_song: For removing the song currently selected

11. load_previous_song():For playing the previous of the currently played song.

CODING THE View.py FILE
Following are the modules to be used in View.py file :
import random
import sys
import threading
import time
from tkinter import filedialog, messagebox
from cx_Oracle import DatabaseError
from pygame import mixer
import tkinter as tk
import tkinter.ttk as ttk
import Player
from MyExceptions import *

1. The __init__() Method: 
This method will do the following:
A. Call the setup_player() method

class Player:

    def __init__(self):
        # do not change any pre written code. Simply add the below mentioned statement
		self.setup_player()


2. The setup_player() Method: 
This method will do the following:
1. It will accept no argument and will be called by the __init__() method as soon as the application loads
2. It will instantiate the Player object
3. It will check whether database connection is opened or not by calling the get_db_status() method of the Player object
4. If the get_db_status() method returns True, then it will show the message "Connected successfully to the database"
5. But if the get_db_status() method returns False, then it will throw an Exception and display the message 	          	    "Sorry! you cannot save or load favourites!!!" . It will also disable the addfavourite, removefavourite and loadfavourite Buttons
6. Finally it will setup the UI elements event handlers as mentioned below:
	a. UI Element: self.vol_scale:
	
Attribute	Value
from_	0
to	100
command	self.change_volume

	b. It will also call the method set() of self.vol_scale passing it the number 50 as argument to set the volume level to 	0.5 initially.
      c. UI Element: self.addToPlayListButton:
	
Attribute	Value
command	self.add_song

	d. UI Element: self.deleteSongsFromPlaylistButton:
	
Attribute	Value
command	self.remove_song

	e. UI Element: self.playButton:
	
Attribute	Value
command	self.play_song

	f. UI Element: self.stopButton:
	
Attribute	Value
command	self.stop_song

	g. UI Element: self.pauseButton:
	
Attribute	Value
command	self.pause_song


	h. UI Element: self.playList:
	
Attribute	Value
font	"Vivaldi 12"

	i. It will also call the method bind() of self.playList for setting up the event handler for double click event
	j. It will also call the method title() and iconphoto() of self.top for setting the title to "MOUZIKKA-Dance to the rhythm of your heart!!" as well as setting up the icon image
	h. It will also call the method protocol() of self.top() passing it 2 arguments. This method allows us to add our own logic before the terminating or destroying the root window. It's first argument is a string called "WM_DELETE_WINDOW" and the second argument is the name of the method to be called when the window is closed by the user.
	i. Finally it will declare 2 instance members called self.isPlaying and self.isPaused and set them to False. These members will be used to implement play and pause functionality.

Based on the above discussion following is the code for the setup_player() method:
def setup_player(self):
    try:
         self.my_player = Player.Player()
         if self.my_player.get_db_status():
             messagebox.showinfo("Success!", "Connected successfully to the DB!!!")
         else:
             raise Exception("Sorry! you cannot save or load favourites!!!")

    except Exception as ex:
        print("DB Error:",ex)
        messagebox.showerror("DB Error!", ex)
        self.addFavourite.configure(state="disabled")
        self.loadFavourite.configure(state="disabled")
        self.removeFavourite.configure(state="disabled")



    self.vol_scale.configure(from_=0, to=100, command=self.change_volume)
    self.vol_scale.set(50)
    self.addSongsToPlayListButton.configure(command=self.add_song)
    self.deleteSongsFromPlaylistButton.configure(command=self.remove_song)
    self.playButton.configure(command=self.play_song)
    self.stopButton.configure(command=self.stop_song)
    self.pauseButton.configure(command=self.pause_song)
    self.playList.configure(font="Vivaldi 12")
    self.playList.bind("<Double-1>", self.list_double_click)
    img = tk.PhotoImage(file="./icons/icon_music1.png")
    self.top.iconphoto(self.top, img)
    self.top.title("MOUZIKKA-Dance to the rhythm of your heart!!")
	self.top.protocol("WM_DELETE_WINDOW", self.closewindow)
    self.isPaused = False
    self.isPlaying = False


3. The change_volume() Method: 
This method will do the following:
A. It will be called in 2 situations :
	a. At the start of application to set the initial volume level
	b. Whenever the user drags the Scale object on the UI
It will accept an argument called volume_level which will be the Scale value passed by Python as soon as the Scale is dragged by the user or we call the method set() of the Scale object.
B. This value will be of type string and in the range of 0 to 100. So it has to be converted to float as well as between 0.0 to 1.0 .
C. Finally to set the volume level we have to call the method set_volume() of the Player object passing it the volume calculated above .
Based on the above discussion following is the code for the change_volume() method:

def change_volume(self,val):
    pass

4. The add_song() Method: 
This method will do the following:
A. This method will be called whenever the user clicks on the      Button.  It will call the method add_song() of the Player object and receive the name of the song in a variable called song_name.
B. If the method add_song() of the Player object returned an empty string , this method would return.
C. Otherwise it will add the song name to the self.playList Listbox object 
D. Finally it will change the color of self.playList to some random color
 Based on the above discussion following is the code for the add_song() method:
def add_song(self):

        pass

5. The show_song_details() Method: 
This method will do the following:
A. This method will be called for showing the song details before the song starts playing. It will accept no argument 
B. It will call the method get_song_length() of the Player object and store the return value in an instance member called self.song_length. We have created song_length as an instance member because we will require it when we will implement the showing song progress functionality using multithreading.
C. It will then convert self.song_length which is in seconds to minutes and seconds and display it in the label self.songTotalDuration . Along with this it will also set the self.songTimePassed to the text "0:0"
D. The next task is to show the song name in the label self.songName. But since the label is of short length , so we will have to truncate the song name to 14 characters if it is of more than 14 letters and add a suffix of . . .
E. Finally we will show the song name in the label self.songName.
Based on the above discussion following is the code for the show_song_details() method:
def show_song_details(self):
    pass



6. The play_song() Method: 
This method will do the following:
A. This method will be called in 2 situations:
	a. When the user selects a song and clicks the  Button
	b. Whenever the users double clicks on a song in the playlist.
	It will accept no argument 
B. It will retrieve the item selected by the user in the playlist . This will be done by calling the method curselection() of the self.playList 
C. If the user has not selected any song , it will raise the NoSongSelectedError exception displaying the message "Please select a song to play".
D. Otherwise it will get the song name from the tuple returned by the method curselection() and store it in an instance member called self.song_name
E. It will then call the instance method self.show_song_details() for displaying song info 
F. Finally it will call the play_song() method of Player object and set the instance member isPlaying to True.
Based on the above discussion following is the code for the play_song() method:
def play_song(self):
    pass





7. The list_double_click() Method: 
This method will do the following:
A. This method will be called as soon as the user double clicks on a song in the playlist . It will accept 1 argument called e of type Event.
B. Since on double click of song in the playlist also we want to play the song , so we will simply call the method play_song() from here

Based on the above discussion following is the code for the list_double_click() method:
def list_double_click(self,e):
   pass


8. The stop_song() Method: 
This method will do the following:
A. This method will be called whenever the user clicks on the  Button. It will stop the song currently being played by calling the method stop_song() of the Player object.
B. It will also set the instance member isPlaying to False.

Based on the above discussion following is the code for the stop_song() method:
def stop_song(self):
    pass

9. The pause_song() Method: 
This method will do the following:
A. This method will be called whenever the user clicks on the  Button and it will pause/unpause the music being currently played.
B. To do this it will call the method pause() and unpause() of the Player object
Based on the above discussion following is the code for the pause_song() method:
def pause_song(self):
    pass

10. The remove_song() Method: 
This method will do the following:
A. This method will be called when the user clicks on the  Button and will remove the song from the playlist which has been selected by the user.It will accept no argument.
B. It will retrieve the item selected by the user in the playlist . This will be done by calling the method curselection() of the self.playList 
C. If the user has not selected any song , it will raise the NoSongSelectedError exception displaying the message "Please select a song to remove".
D. Otherwise it will get the song name from the tuple returned by the method curselection() and store it in a local variable  called song_name. Then it will remove that song from the playlist by calling the method delete() of the Listbox object
E. Finally it will call the method remove_song() of the Player object passing it the song name as argument to remove the entry of this song from the dictionary
Based on the above discussion following is the code for the remove_song() method:
def remove_song(self):
    pass

11. The close_window() Method: 
This method will do the following:
A. As mentioned above , this method will be called by Python when the users clicks on the      Button of the root window.    
B. It will then get a confirmation from the user that whether he wants to quit. This will be done by calling the function askyesno() of the messagebox module.
C. If the user response is True then it will unload the player by calling the method close_player() of the self.my_player object , display a Thank you message using  the function showinfo() of the messagebox module and finally it will destroy the window by calling the method destroy() of self.top.

Based on the above discussion following is the code for the close_window() method:
def closewindow(self):
    pass


12. The load_previous_song() Method: 
This method will do the following:
A. This method will be called when the user clicks on the  Button and will play the song from the playlist which has been selected by the user.It will accept no argument.
B. It will get the index of the song curre
C. If the user has not selected any song , it will raise the NoSongSelectedError exception displaying the message "Please select a song to remove".
D. Otherwise it will get the song name from the tuple returned by the method curselection() and store it in a local variable  called song_name. Then it will remove that song from the playlist by calling the method delete() of the Listbox object
E. Finally it will call the method remove_song() of the Player object passing it the song name as argument to remove the entry of this song from the dictionary
Based on the above discussion following is the code for the load_previous_song() method:
def load_previous_song(self):

    pass


# THE Player.py FILE
The second module that we are going to code is the Player module which will act like Controller. Recall that the Controller controls the flow of information. When we request some data from View then that request is passed to the Controller. The Controller then uses programmed logic to decide what information is needed to be pulled from the Model and passes it to the View
STEPS TO BE DONE BEFORE Player.py
Before writing logic for Player.py , we must download 2 libraries called pygame and mutagen. The pygame library will provide us music playing features like play, stop, pause etc while the mutagen library will provide us internal details of a song called ID3 tag like length of the song, genre, singer, album etc.

To download these libraries from Pycharm , we need to do the following:
Before beginning make sure your internet connection is on and working
1. Click on File-->Settings
2. In the window that appears click on the name of the project on the left pane. In our case it is Project mouzikka


3. Now click on Project Interpreter option which will open a module installer window.


4. Doing this will open another window where we have to type the name of the library we want to download. Type the name pygame and click on Install Package button present at the bottom of the screen.


5. Repeat same steps for downloading the mutagen library
STEPS TO BE DONE IN Player.py
The Player module primarily has 2 tasks :
I. Managing the media player functionality
II. Interacting with the Model class 

So the Player module will do the following:

1. Define a class called Player
2. This class will have the following responsibilities:
	A. Initialize the pygame.mixer module . 
           The pygame.mixer module provides us several functions to handle music playing and management.
      B. Instantiate the Model class so that we can call it's instance methods for interacting with DB and dictionary. 
      C. Provide instance methods for playing ,stopping, pausing and unpausing a song.
      D. Provide instance methods for calling corresponding instance methods of the Model object
         
Based on the above discussion following will be the methods in our Player class:

1. __init__(): For initializing the pygame.mixer module  and instantiating the Model class

2. get_db_status( ): For calling the Model method get_db_status( ) and returning True or False depending on whether the connection to database has been opened or not

3. close_player():For unloading the player and closing the database connection maintained by the Model object

4. set_volume(): For setting the volume level according to the Scale value set by the user.

5. add_song(): For allowing the user to select a song and  adding song entry(song name and song path)  to the song_dict dictionary. 

6. remove_song(): For removing song entry from the song_dict dictionary whose name is passed as argument

7. get_song_length(): For returning the length of the song in seconds whose name is passed as the argument.

8. play_song():For playing the currently selected  song.

9. stop_song(): For stopping the song currently being played.

10. pause_song():For pausing the song currently being played.

11. unpause_song():For unpausing the song currently paused.

12. add_song_to_favourites():For adding song details (song name and song path)  of the song name passed as argument to the database. 

13. load_songs_from_favourites():For loading all the favourite songs from the database and returning them as a dictionary

14. remove_song_from_favourites(): For calling the Model method remove_song_from_favourites() passing it the  song name to be removed from the  database. This method returns the message Song deleted from your favourites or Song not present in your favourites depending on whether the song is removed from the database or not


CODING THE Player.py FILE
Following are the modules to be used in Player.py file :
import Model
from pygame import mixer
from tkinter import filedialog
import os
from mutagen.mp3 import MP3

1. The __init__() Method: 
This method will do the following:
A. Initialize the pygame.mixer module : To do this we have to call the function init() of the module pygame.mixer which initializes the mixer module for Sound loading and playback
B. Declare and initialize an instance variable called my_model of the class Model.Model .
Based on the above discussion following is the code for the __init__() method:

class Player:

    def __init__(self):
        mixer.init()
        self.my_model=Model.Model()

2. The get_db_status() Method: 
This method will do the following:
1. It will accept no argument
2. It will call the method get_db_status()  of the Model object and return it's value
Based on the above discussion following is the code for the get_db_status() method:
def get_db_status(self):
    return self.my_model.get_db_status()


3. The close_player() Method: 
This method will do the following:
A. Unload/stop the music : To stop the music we have to call the function stop() of the module music in the module mixer.
B. Close the connection to the DB: This can be done by calling the method close_db_connection() of the Model object 
Based on the above discussion following is the code for the close_player() method:

def close_player(self):
    mixer.music.stop()
    self.my_model.close_db_connection()

4. The set_volume() Method: 
This method will do the following:
A. It will accept an argument called volume_level which will be the Scale value passed by the View module.
B. Then it will call the function set_volume() of the module mixer.music passing it the volume_level as argument.

Based on the above discussion following is the code for the set_volume() method:
def set_volume(self,volume_level):
    mixer.music.set_volume(volume_level)

5. The add_song() Method: 
This method will do the following:
A. It will ask the user to choose a song file by calling the function filedialog.askopenfilename(). It will make sure that the user is only able to select mp3 files.
B. If the user didn't select any file then the method will return.
C. Otherwise it will extract the name of the song name from the complete song path returned by the function filedialog.askopenfilename(). This is done by calling the function os.path.basename(). This function belongs to the path sub module of os module and accepts a file path as argument and returns the file name.
D. It will then add the song name and song path as key-value pair to the dictionary in Model object by calling the method add_song() of Model object.
E. Finally it will return the song name 
Based on the above discussion following is the code for the add_song() method:
def add_song(self):
    song_path = filedialog.askopenfilename(title="Select your song. . .",filetypes=[("mp3 files",".mp3")])
    if song_path=="":
            return
    song_name = os.path.basename(song_path)
    self.my_model.add_song(song_name,song_path)
    return song_name

6. The remove_song() Method: 
This method will do the following:
A. It will accept 1 argument called song_name.
B. It will call the method remove_song() of the Model object to remove the entry of this song from the instance variable song_dict 
Based on the above discussion following is the code for the remove_song() method:
def remove_song(self,song_name):
    self.my_model.remove_song(song_name)


7. The get_song_length() Method: 
This method will do the following:
A. It will accept 1 argument called song_name.
B. It will then get the complete path of the song by calling the method get_song_path() of the Model object.
C. Then it will create an object of the class MP3 of the module mutagen passing the song path to it's constructor. 
D. This object contains another object called info which has a data member called length which returns the length of the song in seconds.
E. Finally it will return the length of the song.

Based on the above discussion following is the code for the get_song_length() method:
def get_song_length(self,song_name):
    self.song_path = self.my_model.get_song_path(song_name)
    self.audio_tag = MP3(self.song_path)
    song_length = self.audio_tag.info.length
    return song_length

8. The play_song() Method: 
This method will do the following:
A. It will accept no arguments
B. It will at first uninitialize the mixer by calling the function quit() of mixer module.
C. Then it will initialize the mixer by calling the function init() of the mixer module . It will also pass the playback speed as argument to this method so that the music gets played exactly at the same speed as it should. To do this we can pass a keyword argument called frequency_rate to the function init() . The value passed to this argument the data member sample_rate of the object info in the object of MP3
D. Then it will load the song by calling the load() of the mixer.music module
E. Finally it will start playing the music by calling the function play() of mixer.music

Based on the above discussion following is the code for the play_song() method:
def play_song(self):
    pass


9. The stop_song() Method: 
This method will do the following:
A. It will stop the music being currently played.
B. To do this it will call the function stop() of the sub module music of the module mixer

Based on the above discussion following is the code for the stop_song() method:
def stop_song(self):
    pass


10. The pause_song() Method: 
This method will do the following:
A. It will pause the music being currently played.
B. To do this it will call the function pause() of the sub module music of the module mixer

Based on the above discussion following is the code for the pause_song() method:
def pause_song(self):
    pass

11. The unpause_song() Method: 
This method will do the following:
A. It will unpause the music which has been paused.
B. To do this it will call the function unpause() of the sub module music of the module mixer

Based on the above discussion following is the code for the unpause_song() method:
def unpause_song(self):
    pass

12. The add_song_to_favourites() Method: 
This method will do the following:
A. It will accept 1 argument called song_name 
B. It will retrieve the path to the song by calling the method get_song_path() of the Model object.
C. It will then call the method add_song_to_favourites() of the Model object passing it the song name and song path as argument.
D. Finally it will receive the return value of add_song_to_favourites() and return it 

Based on the above discussion following is the code for the add_song_to_favourites() method:
def add_song_to_favourites(self,song_name):
    pass

13. The load_songs_from_favourites() Method: 
This method will do the following:
A. It will call the method load_songs_from_favourites()of the Model object
B. it will receive the return value of load_songs_from_favourites() and return this value as well as the song_dict of the Model object

Based on the above discussion following is the code for the load_songs_from_favourites() method:
def load_songs_from_favourites(self):
    pass


14. The remove_song_from_favourites() Method: 
This method will do the following:
A. It will accept 1 argument of called song name
B. It will call the method remove_song_from_favourites() of the Model object passing it the song name as argument
C. It will receive the return value of load_songs_from_favourites() and return this value 

Based on the above discussion following is the code for the remove_song_from_favourites() method:
def remove_song_from_favourites(self,song_name):
	pass



# Model 
THE Model.py FILE
The first module that we are going to code is the Model module. This is because all other 2 modules(View and Player) will depend on this module for most of their functionality.

STEPS TO BE DONE IN Model.py
The Model module primarily has to do following tasks:
1. Define a class called Model
2. This class will have the following responsibilities:
	A. Maintain a collection of songs selected by the user to play . 
           This collection should hold a pair of song name and song path and so it will be a dictionary and we'll name it   		    song_dict
      B. Provide methods for adding and removing song entry(song name and song path) to this collection as well as 	 	    retrieving song path corresponding  to a given song name  from this collection.
      C. Perform database interaction 
           Provide methods for opening , closing the database connection
          Also provide methods for adding , removing ,searching and loading songs from  favourites
Based on the above discussion following will be the methods in our Model class:

1. __init__(): For initializing the song dictionary and opening the database connection. The instance variables used will be called song_dict , conn and cur

2. get_db_status( ): Returning True or False depending on whether the connection to database has been opened or not

3. close_db_connection():For closing the database connection 

4. add_song(): For adding song entry(song_name and song_path)  passed as argument to the song_dict dictionary. 

5. get_song_path(): For returning the song_path from the song_dict dictionary whose song_name is passed as argument

6. remove_song(): For removing song entry from the song_dict dictionary whose song_name is passed as argument

7. search_song_in_favourites(): For searching the song in the database whose name is passed as argument . This method returns True if the song is found in database, otherwise it returns False
 
8. add_song_to_favourites():For adding song entry(song name and song path)  passed as argument to the database.

9. load_songs_from_favourites(): For searching  the database and loading all the songs in the dictionary song_dict.

10. remove_song_from_favourites(): For removing the song from the database whose name is passed as argument . This method returns the message Song deleted from your favourites or Song not present in your favourites depending on whether the song is removed from the database or not


CODING THE Model.py FILE
1. The __init__() Method: 
This method will do the following:
A. Declare an instance variable called song_dict of type dictionary.
B. Declare an instance variable called db_status of type boolean initially set to True.
C. Declare an instance variable called conn of type Connection initially set to None.
D. Declare an instance variable called cur of type Cursor initially set to None.
E. Attempt to open the connection to the database by calling the function connect() of cx_Oracle module with the credentials mouzikka/music. 
F. If the connection is successful it will also initialize the instance variable cur to the Cursor object otherwise it will set the instance variable db_status to False

Based on the above discussion following is the code for the __init__() method:

class Model:

    def __init__(self):
        self.song_dict={}
        self.db_status = True
        self.conn = None
        self.cur = None
        try:

            self.conn = connect("mouzikka/music@Sachin-PC/orcl")
            print("Connected successfully to the DB")
            self.cur = self.conn.cursor()
        
		except DatabaseError:
            self.db_status = False

2. The get_db_status() Method: 
This method will do the following:
A. It will accept no argument
B. It will return the value of the instance variable db_status

Based on the above discussion following is the code for the get_db_status() method:
def get_db_status(self):
        return self.db_status


3. The close_db_connection() Method: 
This method will do the following:
A. Check whether the instance variable cur is None or not . If it is not None , then it will be closed.
B. Check whether the instance variable conn is None or not . If it is not None , then it will be closed.

Based on the above discussion following is the code for the close_db_connection() method:

def close_db_connection(self):
    if self.cur is not None:
        self.cur.close()
        print("Cursor closed successfully")
    if self.conn is not None:
        self.conn.close()
        print("Disconnected successfully from the DB")

4. The add_song() Method: 
This method will do the following:
A. It will accept two arguments called song_name and song_path.
B. It will add them as key-value pair in the instance variable song_dict where song_name will be the key and song_path will be the value.

Based on the above discussion following is the code for the add_song() method:
def add_song(self,song_name,song_path):
    self.song_dict[song_name] = song_path
    print("song added:",self.song_dict[song_name])


5. The get_song_path() Method: 
This method will do the following:
A. It will accept 1 argument called song_name.
B. It will return the song path  of this song by retrieving it from the instance variable song_dict

Based on the above discussion following is the code for the get_song_path() method:
def get_song_path(self,song_name):
    return self.song_dict[song_name]

6. The remove_song() Method: 
This method will do the following:
A. It will accept 1 argument called song_name.
B. It will remove the entry of this song from the instance variable song_dict

Based on the above discussion following is the code for the remove_song() method:
def remove_song(self,song_name):
    self.song_dict.pop(song_name)
    print(self.song_dict)

7. The search_song_in_favourites() Method: 
This method will do the following:
A. It will accept 1 argument called song_name.
B. Search the database to check whether the song_name is present in the table Myfavourites or not.
C. If it is present it will return True otherwise it will return False

Based on the above discussion following is the code for the search_song_in_favourites() method:
def search_song_in_favourites(self,song_name):

    pass


8. The add_song_to_favourites() Method: 
This method will do the following:
A. It will accept 2 arguments called song_name and song_path
B. Search the database to check whether the song_name is present in the table Myfavourites or not.
C. If it is present it will return the message Song already present in your favourites.
D. If the song is not present in the table , then it will generate the song id for the song . To do this it will find the song_id of the last song from the table and increment it by one to get the next song id.
E. Finally it will insert a new record in the table Myfavourites with the generated song_id and the song_name and       song_ path and return the message Song added to your favourites

Based on the above discussion following is the code for the add_song_to_favourites() method:
def add_song_to_favourites(self,song_name,song_path):
    pass

9. The load_songs_from_favourites() Method: 
This method will do the following:
A. It will accept no arguments
B. Retrieve all the rows from the table Myfavourites with the columns song_name and song_path.
C. Run a loop to retrieve song_name and song_path of each row and add it to the instance variable song_dict as 		key-value pair.
D. If the songs are added to the dictionary it will return the message List populated from favourites  otherwise it will return the message No songs present in your favourites 

Based on the above discussion following is the code for the load_songs_from_favourites() method:
def load_songs_from_favourites(self):

    pass 

10. The remove_song_from_favourites() Method: 
This method will do the following:
A. It will accept 1 argument called song_name
B. Execute the delete query to remove the given song from the table Myfavourites.
C. If the song is deleted it will also remove it from the instance variable song_dict and return the message Song deleted from your favourites otherwise it will return the message Song not present in your favourites .

Based on the above discussion following is the code for the remove_song_from_favourites() method:
def remove_song_from_favourites(self,song_name):
    pass
