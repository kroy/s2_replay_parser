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

Structure of a Replay file:

	An s2 replay file (s2r2) is a binary file containing a compressed representation of all of the game data
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
			-> Not sure what the purpose of separating the two functions out like this is.
