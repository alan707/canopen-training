CANOpen Training
-----------------------------------

### Excercise 1

Install can-utils

    $ sudo apt-get update
    $ sudo apt-get install can-utils

### Clone CANOpenSocket Repo


### Excercise 2

### Clone CANOpenSocket Repo

Clone the project from git repository and get submodules:

    $ git clone https://github.com/CANopenNode/CANopenSocket.git
    $ cd CANopenSocket
    $ git submodule init
    $ git submodule update


### Download CANOpen GUI Application



### Blue and Red Teams: Test CAN dump

Set up a virtual CAN device:

    $ sudo modprobe vcan
    $ sudo ip link add dev vcan0 type vcan
    $ sudo ip link set up vcan0
    
You should see it in ifconfig.  Run:

    $ ifconfig

If you haven't yet, install `can-utils`  

    $ sudo apt-get install can-utils


### Blue Teams: Set up Node 2 device

From terminal, compile and start *canopend*.

    $ cd CANopenSocket/canopend
    $ make
    $ app/canopend --help
    $ echo - > od4_storage
    $ echo - > od4_storage_auto
    $ app/canopend vcan0 -i 32 -s od4_storage -a od4_storage_auto

In a separete terminal, now run: 

    candump vcan0
    
And you should see an output similar to this:

    vcan0  704   [1]  00                        # Bootup message.
    vcan0  084   [8]  00 50 01 2F F3 FF FF FF   # Emergency message.
    vcan0  704   [1]  7F                        # Heartbeat messages
    vcan0  704   [1]  7F                        # one per second.


Now there is operational state (0x05) and there shows one PDO on CAN
address 0x184. To learn more about PDOs, how to configure communication
and mapping parameters and how to use them see other sources of CANopen
documentation (For example article of PDO re-mapping procedure in [CAN
newsletter magazine, June 2016](http://can-newsletter.org/engineering/engineering-miscellaneous/160601_can-newsletter-magazine-june-2016) ).

### Red Teams: Set up Node 2 device

Start *canopend* (master on nodeID=2) in the same

    $ app/canopend vcan0 -i 2 -c "/home/$USER/canopensocket"




Compile and start canopencomm.

    $ cd CANopenSocket/canopencomm
    $ make
    $ ./canopencomm --help

#### SDO master

Play with it and also observe CAN dump terminal. First Heartbeat at
index 0x1017, subindex 0, 16-bit integer, on nodeID 32.

    $ ./canopencomm [1] 32 read 0x1017 0 i16 -s "/home/$USER/canopensocket"
    $ ./canopencomm [1] 32 write 0x1017 0 i16 5000 -s "/home/$USER/canopensocket"

In CAN dump you can see some SDO communication. You will notice, that
Heartbeats from node 32 are coming in 5 second interval now. You can do
the same also for node 2. Now store Object dictionary, so it will preserve
variables on next start of the program.

    $ ./canopencomm 32 w 0x1010 1 u32 0x65766173 -s "/home/$USER/canopensocket"

You can read more about Object dictionary variables for this
CANopenNode in [canopend/CANopenSocket.html].


#### NMT master
If node is operational (started), it can exchange all objects, including
PDO, SDO, etc. In pre-operational, PDOs are disabled, SDOs works. In stopped
only NMT messages are accepted.

    $ ./canopencomm 32 preop -s "/home/$USER/canopensocket"
    $ ./canopencomm 32 start -s "/home/$USER/canopensocket"
    $ ./canopencomm 32 stop -s "/home/$USER/canopensocket"
    $ ./canopencomm 32 r 0x1017 0 i16 -s "/home/$USER/canopensocket"		# time out
    $ ./canopencomm 32 reset communication -s "/home/$USER/canopensocket"
    $ ./canopencomm 32 reset node -s "/home/$USER/canopensocket"
    $ ./canopencomm 2 reset node -s "/home/$USER/canopensocket"

In *canopend terminal* you see, that both devices finished. You will need to manually start up `Node 32` and `Node 2`
