# RLBot over TCP

![TCP RLBot diagram](rlbot.png)

This is an experimental implementation of RLBot 
that communicates with bot processes via TCP sockets. 
In addition to bots, RLBot can also connect to a
controller process that manages match execution, 
state setting, and sending custom
messages to bots (e.g. for training exercises).

The messages sent over TCP support a JSON format,
offering new developers a simple way to get up and running,
and a flatbuffer format (not implemented yet) for veterans 
that are more interested in optimizing the performance of 
their bot.

Each time RLBot receives a packet from Rocket League, it starts a
timer for about 6 milliseconds (for 120 Hz). After processing the 
contents of the packet from Rocket League, RLBot sends a packet of 
updated game information to each of the connected bots. Once the 
timer expires, RLBot gathers the responses from each of the bots
and assembles a packet of messages (e.g. the inputs for each car,
state setting, rendering) to send off to Rocket League. Bots that
take longer than the allotted time will not receive new messages
from RLBot until a response is received.

## How do I write a custom Rocket League bot?

The basic steps (and pseudocode) are:
1. Connect to RLBot on the specified port (default is 23235)
```cpp
tcp::socket socket;
socket.connect(port);
```

2. Perform any necessary setup, and send RLBot a `Ready` message
to start receiving data.
```cpp
// do any expensive calculations and setup
reticulate_splines()

socket.write(serialize(Ready{
  my_id,
  my_team,
  my_name,
  my_loadout
});
```

3. Read the information from RLBot about the current state of the game,
and decide what to do. 
```cpp

// the main loop
while (true) {

  // receive information from RLBot
  auto messages = socket.read_packet();

  // analyze the new game state and other messages
  process(messages);

  // figure out what to do
  auto controls = calculate_optimal_controls();

  // optionally draw some stuff 
  auto rendering_command = …;

  // send our messages back to RLBot
  socket.write(serialize({controls, rendering_command})); 

}
```

In theory, any language that supports TCP communication should be
able to work with RLBot! Click [here](https://github.com/samuelpmish/RLBot/wiki/JSON-Message-Specification)
for more information about the different message types RLBot supports.
