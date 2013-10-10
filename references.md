Bitarray: https://pypi.python.org/pypi/bitarray/
Structs: http://docs.python.org/2/library/struct.html
Struct Format chars
		x	pad byte	no value	 	 
		c	char	string of length 1	1	 
		b	signed char	integer	1	(3)
		B	unsigned char	integer	1	(3)
		?	_Bool	bool	1	(1)
		h	short	integer	2	(3)
		H	unsigned short	integer	2	(3)
		i	int	integer	4	(3)
		I	unsigned int	integer	4	(3)
		l	long	integer	4	(3)
		L	unsigned long	integer	4	(3)
		q	long long	integer	8	(2), (3)
		Q	unsigned long long	integer	8	(2), (3)
		f	float	float	4	(4)
		d	double	float	8	(4)
		s	char[]	string	 	 
		p	char[]	string	 	 
		P	void *	integer	 	(5), (3)

Terminology:
	
	Entity: the basic building blocks of a HoN game. Everything in a HoN game is done to an entity, and is caused by
		an entity (I believe). In the context of the replay file, Entities are represented as objects with a certain
		number of fields. There is a significant amount of magic involved in parsing entity objects.

Classes included in s2r2.py and their purposes:
	
	ReplayManager: This class initializes and builds the replay data into an intelligible format using the other classes
		included in s2r2.py.  It allows the user to step through each frame of the replay and extract all entity states.
		The StartPlayback() method must be called before stepping through each frame with NextFrame().

	BitBuffer: A wrapper for a bitarray. This allows for bits to be read from the replaydata in a special way in order
		to respect the format of the s2 replay data

	EntitySnapshot: Represents the state of an entity at a given point in time

	CSnapshot: Represents the data contained within a frame of the replay. The data includes entities and game events


Structure of a Replay file:

	An s2 replay file is a binary file containing a representation of all of the game data
	included in a HoN game.  The game is divided into frames which contain all data for a given snapshot of the game.
	Frames contain references to entities, which are the building blocks of a replay.

	-The first 4 bytes of data in the replay file should be the string: 'S2R2'
	-The next 4 bytes are an int representing the format version of the replay data
	-The next 4 bytes are the server version on which the game was played
	-The next piece of the replay file is (I believe) a set of variables which pertain to the
		initialization of the map and game
			-> In order to understand these, need to understand the EntitySnapshot class
			-> These variables are divided into objects, which I believe are called "entities"
			-> The initial values for each of the entity's vars is stored in the chunk next to 
				the var descriptions
			-> Combining the var descriptions with the initial values gives us a snapshot of the entity
	-The next chunk of data is a mysterious array of vars whose value is a short
	-Next is a string representing the map name
	-Next is the extremely important "StringSet".  This is an array of dictionaries, each one holding the values
		of certain pieces of important game info dependent on the version of the replay.
			-> The size is read in as a variable
			-> For each entry in the StringSet, we parse the contents of the string into a dict with format
				key : value
			-> dict 0 (StringSets[0]) contains information about the server on which the game was hosted
			-> dict 1 (StringSets[1]) contains some global vars affecting game mechanics
			-> dict 2 (StringSets[2]) contains some mystery vars, not sure what they are
			-> dict 3 (StringSets[3]) is very important. It contains the names of each entity in this version of the
				game, keyed by id number.  These id numbers are referenced by other parts of the replay data to 
				convey an action has been performed on a specific entity
					example: '593': '064/heroes/drunkenmaster/ability_03/projectile.entity'
	-Next are the StateBlocks which are used to build the EntityMap. The EntityMap is what links the EntityPool
		to the StringSets[3] dictionary. This is how the name of the entity in question is extracted from the raw
		entity tuple contained in the entity pool.
			-> I am not sure what the purpose of including the Map in the first place.  It seems that it would be easier
				to simply include the StringSet[3] id in the EntityPool tuple.

	--------------------End of Replay Data initialization (all of the above is parsed in ReplayManager.StartPlayback())

	-The next chunk of data contains the frames of the actual replay.  Each frame is structured as follows
		-> The first 4 bytes of a frame contain the length of the frame
		-> The replay parser reads the frame into a BitBuffer.
		-> The BitBuffer is parsed into a snapshot
