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
