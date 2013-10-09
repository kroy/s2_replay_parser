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
	-The next piece of the replay file are (I believe) variables which pertain to the
		initialization of the map and game
