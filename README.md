An audio streaming application that operates using C++, the irrKlang library and the TCP/IP stack

DISCLAIMER:
This project is inspired by Jimmie Johnsson's tutorial ([link](https://www.youtube.com/watch?v=VrOZ9BiWd-8&ab_channel=JimmieJohnsson)). The primary goal of this project is to gain a deeper understanding of using C++ over the TCP/IP stack, learn how to utilize the irrKlang library for music processing, and further develop my C++ skills.
I have created my own documentation for this project to better understand how the application will operate and to structure it as an Agile project. The first objective is to create a Minimum Viable Product (MVP) for the initial iteration.
I am not copying and pasting any code; I will be implementing everything myself, following the architecture detailed in the tutorial. Special thanks to Jimmie Johnsson for providing such an informative video that served as a valuable learning resource.


Problem Definition:

	1. A Server that can stream music to clients
		> Needs to handle clients
		> Needs to send music
		> Needs to send cover art 
	2. A client that can request and play back the streamed music
		> Needs to connect to server
		> Needs to use systems audio device
		> Need to handle user input for new song request
		> Needs to handle disconnections
		> Needs to play audio
	3. Need a music library
		> Use local files 


Analysis/Key Features

Linux

The application will utilise a Fork operator to enable multiple clients connecting. This specific operator is only compatible with Linux architecture. This will be achieved by using the VM WSL2

TCP/IP

The TCP/IP stack is a set of protocols used for data to traverse the internet. It requires data to be converted to byte data. We are using TCP over UDP here as we want data to go omni-directionally where UDP is only used for single-direction traffic. Whilst it would be possible to use the faster protocol of UDP exclusively for transmitting song data from the server to the client, this is out of the scope for this project at this time. 

Serialisation

Serialisation is process of converting data into a format can be transmitted. This can be done a variety of ways however in this instance we will be converting into binary - one of the most efficient methods.

Sockets

Sockets are a memory buffer within a file system through which a process can write data to. They allow for bi-directional communication and will be the way our client communicate with the server. The server will continuously check the socket to see if a new packet has been received or that there is one to send. Each client connection will have their own socket.

Multithreading

Multithreading is the process of simultaneously processing multiple sections of code. This is achieved by creating multiple threads that run simultaneously.

Mutex

Mutex / mutual exclusion prevents multiple threads from accessing or modifying a shared variable/section of memory/resource at the same time. In C++ two ways this can be achieved is through using either a lock_guard or unique_lock. Lock_guard automatically protects the critical section of the code until the lock object goes out of scope whereas unique_lock needs to be manually unlocked but can be more versatile.





Server Design

		> Will use a fork structure to handle multiple clients.
    > Main will listen for any new connections and create a child process to handle them.
    > Parent process will handle closing child process once user disconnects.
    > Will need to tell parent process to ignore child process using signal.
  
![image](https://github.com/user-attachments/assets/b347ef86-9064-478d-9bd6-edacaa773ced)

Client Design

	> Client application will operate over 3 threads
	> Will launch in main, initialise audio device, connect to server and wait input from user
	> When user requests a new song sends appropriate request on thread network
		○ Server will then begin sending song packets
	> Network receives packets
	> Packet handling copies music packets to irrKlang (C++ library for processing music)
	> Music will begin playing once a certain data threshold has been reached / received
 
![image](https://github.com/user-attachments/assets/a5ff129b-d443-4436-a2c1-99958e7bbecd)

Class Diagrams

Packet

	> Enum class called packetType so that it is easier to read / understand what type of packet it is
		○ Enum classes are a set of names integer constants
	> Packet structure with a type, the contents of the data and packet size
    ○ The size is important as it will allow us to know how much of the song has been recieved 
    
![image](https://github.com/user-attachments/assets/988ee332-ad15-47bf-9b3d-7492b2b76892)

Network Traffic

	> Vectors holding data to receive and to send
	> Mutex variables for each vector to ensure that they are not updated simultaneously by different threads (leading to unexpected/negative outcomes)

![image](https://github.com/user-attachments/assets/4171d8cb-2b92-44f4-b0b4-ebd2a61f77ec)

StreamingMusic

	> Operates of the irrKlang library
	> Irrklang::Isound is a class that represents a sound which is currently played
		○ #include <ik_Isound.h>
	> Irrklang::ik_c8 is an 8 bit character variable
		○ This attribute be a unique pointer to the character variable
		○ A unique pointer is a smart pointer that manages the lifetime of a dynamically allocated object
			§ Object will be automatically destroyed when pointer goes out of cope
	> Irrklang::ik_s32 is an 32 bit signed variable
	> soundDataCreated is a boolean that states if the sound source has been added to the irrklang sound engine (s_engine)


	> setSongName() sets the name of the song
	> startFetchAudio() loads the default values for a new song into the objects attributes
	> fetchAudioToMemory() adds a sound source to the irrklang s_engine
	> startMusic() makes the s_engine begin playing sound
	> stopMusic() removes the sound source from the s_engine
	> readyToPlayMusic() returns a boolean value and checks that a minimum amount of song data has been received

![image](https://github.com/user-attachments/assets/03a95f18-51e2-4cac-b427-25a3c0e4cb2e)






