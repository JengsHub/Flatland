This document describes the data structures and classes that make up a Flatland world.

# Rail map datastructure
A Flatland map is stored as a 2D array of bit patterns representing valid transitions. 

This map is stored in a RailEnv object (found in flatland.envs.rail_env), as part of the env.rail property.
The rail itself is a GridTransitionMap (which is found in flatland.core.transition_map).
The rail contains a grid property, a 2D numpy array of 16-bit integers. 

16386 is a curve that connects south-to-east:
      
        [[    0     0     0  8192     0]
         [    0     0 16386  2064     0]
         [    0 16386  2064     0     0]
         [16386  2064     0     0     0]
         [  128     0     0     0     0]]

    
What is less obvious to see is what the bits of the integer encode. Taking the same tile with encoding 16386, we see that the 16-bit binary value is 0100000000000010. We split this binary string into four 4-bit components matching the N, E, S, W 'incoming' directions. Each 4-bit component again matches the N, E, S, W directions, but now for the 'outgoing' direction. Thus, for this tile, it is possible to come in going North, and exit going East. Or enter heading West, exit going South:
      
        Incoming:     N    |    E    |    S    |    W
        Outgoing:  N E S W | N E S W | N E S W | N E S W 
        Possible?  0 1 0 0 | 0 0 0 0 | 0 0 0 0 | 0 0 1 0   (= 16386)

    
Checking the same pattern with the dead-end tile at the top (3,0) -> 8192, we observe that the only possibility is entering heading North and exiting heading South, which is consistent with the knowledge that we can turn around at dead-ends:
      
        Incoming:     N    |    E    |    S    |    W
        Outgoing:  N E S W | N E S W | N E S W | N E S W 
        Possible?  0 0 1 0 | 0 0 0 0 | 0 0 0 0 | 0 0 0 0   (= 8192 in little endian)

    
# Agent datastructure
In adition to the rail map, the RailEnv is also responsible for tracking the agents. 
The env.agents property stores a List of EnvAgent objects (found in flatland.envs.agent_utils). An EnvAgent has:

  an initial_position tuple recording the (y,x) coordinate the train will enter the map. The red train in the example is thus at initial_position of (0,3) (and not (3,0) as you might expect).
  
  a target tuple recording the (y',x') destination coordinate representing the station the train is trying to reach.
  
  a position, an (y,x) tuple indicating the current position on the rail map,
  
  a direction attribute recording the current heading of the train. It's a Grid4TransitionsEnum from flatland.core.grid.grid4 containing NORTH, EAST, SOUTH, WEST.
  
  a status attribute recording the current status of the train, a RailAgentStatus from the same file. It's an enumerator with four properties:
  
    READY_TO_DEPART: train not yet on the grid, enters at initial_position upon MOVE_xxx,
    ACTIVE train is on the grid at position,
    DONE train reached destination, at position,
    DONE_REMOVED train reached destination, removed from the grid.
    
  a moving boolean which is true if the train is currently moving.

# Agent actions
Every step, every agent should be given an action to take. The action is one of five options given in the RailEnvActions enum in rail_env, namely:

    Action DO_NOTHING, which continues moving if moving = true, but stops if an obstacle is encountered. A fork in the road that does not allow proceeding in the current direction counts as an obstacle.
    Action MOVE_LEFT, which moves left at junctions where the train can divert left w.r.t. its orientation. So if the train is moving EAST, left would be NORTH.
    Action MOVE_FORWARD, which starts moving if the train was stopped and the square is free. Forward is in the direction of the train, but if the tile is a simple curve, it will still change direction!
    Action MOVE_RIGHT, symmetric with LEFT but now for junctions with a right arc.
    Action STOP_MOVING, stops the train (moving = false) if it was moving previously.

There is one special effect to the MOVE_xxx actions. If the train is in status = READY_TO_DEPART, the use of a MOVE_xxx action places the train in the environment (at which point it becomes an obstacle for other trains). However, this action does not start the train's movement (moving = False).

# Solution schedules
To solve the simple map shown above, we therefore need to produce the following schedule:

      
     [ MOVE_FORWARD,    (=> status: READY_TO_DEPART -> ACTIVE, position: None -> (0, 3))
       MOVE_FORWARD,    (=> moving: False -> True, position: (0, 3) -> (1, 3))
       DO_NOTHING,      (=> position = (1, 2))
       DO_NOTHING,      (=> position = (2, 2))
       DO_NOTHING,      (=> position = (2, 1)) 
       DO_NOTHING,      (=> position = (3, 1))
       DO_NOTHING,      (=> position = (3, 0))
       DO_NOTHING       (=> position: (3, 0) -> (4, 0), 
                            status:   ACTIVE -> DONE_REMOVED,
                            position: (4, 0) -> None ) ]
