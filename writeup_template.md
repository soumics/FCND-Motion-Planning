## Project: 3D Motion Planning
![Quad Image](./misc/enroute.png)

---


# Required Steps for a Passing Submission:
1. Load the 2.5D map in the colliders.csv file describing the environment.
2. Discretize the environment into a grid or graph representation.
3. Define the start and goal locations.
4. Perform a search using A* or other search algorithm.
5. Use a collinearity test or ray tracing method (like Bresenham) to remove unnecessary waypoints.
6. Return waypoints in local ECEF coordinates (format for `self.all_waypoints` is [N, E, altitude, heading], where the drone’s start location corresponds to [0, 0, 0, 0].
7. Write it up.
8. Congratulations!  Your Done!

## [Rubric](https://review.udacity.com/#!/rubrics/1534/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it! Below I describe how I addressed each rubric point and where in my code each point is handled.

### Explain the Starter Code

#### 1. Explain the functionality of what's provided in `motion_planning.py` and `planning_utils.py`
These scripts contain a basic planning implementation that includes...

The first basic difference between 'backyard_flyer.py' ['bkf.py'] and 'motion_planning.py' [mp.py] is in creating the waypoints. The bkf.py generates the waypoints using a calculate_box() function/method and simpler, whereas the mp.py uses a_star() function/method to generate the waypoints using the optimization of the cost function which is a summation of heuristic and action costs [computed over all feaasible waypoints/nodes in the graph traversal]. the 'planning_utils.py' [pu.py] assists the mp.py with the implementation of the important functions/ methods like the a_star function/method [function -> independently defined, method -> memeber function of a class]

And here's a lovely image of my results (ok this image has nothing to do with it, but it's a nice example of how to include images in your writeup!)
![Top Down View](./misc/high_up.png)

Here's | A | Snappy | Table
--- | --- | --- | ---
1 | `highlight` | **bold** | 7.41
2 | a | b | c
3 | *italic* | text | 403
4 | 2 | 3 | abcd

### Implementing Your Path Planning Algorithm

#### 1. Set your global home position
Here students should read the first line of the csv file, extract lat0 and lon0 as floating point values and use the self.set_home_position() method to set global home. Explain briefly how you accomplished this in your code.

first way:

(actually this is already written in the faq section!)

        with open('colliders.csv', newline='') as f:
            reader = csv.reader(f)
            row1 = next(reader)  # gets the first line
        lat0 = float(row1[0].split()[1])
        lon0 = float(row1[1].split()[1])
        print('lat0:',lat0,', lon0:',lon0)
        """
        # TODO: set home position to (lon0, lat0, 0)
        """
        self.global_home[0]=lon0
        self.global_home[1]=lat0
        self.global_home[2]=0
        
Second way:

(this is written in the README.md of the project)

        self.global_home[0]=self._longitude
        self.global_home[1]=self._latitude
        self.global_home[2]=0
        print('global home {0}'.format(self.global_home))

Honestly, I have not found a difference!


And here is a lovely picture of our downtown San Francisco environment from above!
![Map of SF](./misc/map.png)

#### 2. Set your current local position
Here as long as you successfully determine your local position relative to global home you'll be all set. Explain briefly how you accomplished this in your code.


        # TODO: retrieve current global position
        global_position=self.global_position
        
        # TODO: convert to current local position using global_to_local()
        local_position=global_to_local(global_position, self.global_home)
        print('local position:',local_position)
        self.local_position[0]=local_position[0]
        self.local_position[1]=local_position[1]
        self.local_position[2]=local_position[2]
        
        print('global home {0}, position {1}, local position {2}'.format(self.global_home, self.global_position,
                                                                         self.local_position))

The first #TODO, I have found from README.md
The second #TODO is an excercise from the tutorial!



Meanwhile, here's a picture of me flying through the trees!
![Forest Flying](./misc/in_the_trees.png)

#### 3. Set grid start position from local position
This is another step in adding flexibility to the start location. As long as it works you're good to go!

        grid_start = (int(self.local_position[0]-north_offset),int(self.local_position[1]-east_offset))
        
Honestly, I have found this from faq section without the 'self.***', but I have understood what does this mean! It is like adding a 'bias' term to the 'true' sensor measurement!

#### 4. Set grid goal position from geodetic coords
This step is to add flexibility to the desired goal location. Should be able to choose any (lat, lon) within the map and have it rendered to a goal location on the grid.

        # TODO: adapt to set goal as latitude / longitude position and convert
        goal_north=100
        goal_east=-150
        grid_goal = (-north_offset + goal_north, -east_offset + goal_east)
        
Performed this #TODO in the same line as that of the setting of the start location, described previously!

#### 5. Modify A* to include diagonal motion (or replace A* altogether)
Minimal requirement here is to modify the code in planning_utils() to update the A* implementation to include diagonal motions on the grid that have a cost of sqrt(2), but more creative solutions are welcome. Explain the code you used to accomplish this step.

        NORTHEAST=(1, 1, 1.4142135623730951)
        NORTHWEST=(-1, 1, 1.4142135623730951)
        SOUTHEAST=(1, -1, 1.4142135623730951)
        SOUTHWEST=(-1, -1, 1.4142135623730951)
        
added this to the Action class for diagonal grid-cell movement, with the cost -> np.sqrt(2)=1.4142135623730951
        
        if (x + 1 < 0 and y + 1 < 0) or grid[x + 1, y + 1] == 1:
            valid_actions.remove(Action.NORTHEAST)
        if (x - 1 < 0 and y + 1 < 0) or grid[x - 1, y + 1] == 1:
            valid_actions.remove(Action.NORTHWEST)
        if (x + 1 < 0 and y - 1 < 0) or grid[x + 1, y - 1] == 1:
            valid_actions.remove(Action.SOUTHEAST)
        if (x - 1 < 0 and y - 1 < 0) or grid[x - 1, y - 1] == 1:
            valid_actions.remove(Action.SOUTHWEST)
            
added this to validate whether the quad is going out off the grid [the first (...) part in 'if'] or whether the diagonal cell is an obstacle [grid[...]==1 part]

the rest of the algorithm operates on the valid actions, and so there is nothing to change!

#### 6. Cull waypoints 
For this step you can use a collinearity test or ray tracing method like Bresenham. The idea is simply to prune your path of unnecessary waypoints. Explain the code you used to accomplish this step.

        def heuristic(position, goal_position):
            return np.linalg.norm(np.array(position) - np.array(goal_position))
            
        def point(p):
            return np.array([p[0], p[1], 1]).reshape(1, -1)

        def collinearity_check(p1, p2, p3, epsilon=1e-6):   
            m = np.concatenate((p1, p2, p3), 0)
            det = np.linalg.det(m)
            return abs(det) < epsilon
        # We're using collinearity here, but you could use Bresenham as well!
        def prune_path(path):
            pruned_path = [p for p in path]
            # TODO: prune the path!
            
            i = 0
            while i < len(pruned_path) - 2:
                p1 = point(pruned_path[i])
                p2 = point(pruned_path[i+1])
                p3 = point(pruned_path[i+2])
                
                # If the 3 points are in a line remove
                # the 2nd point.
                # The 3rd point now becomes and 2nd point
                # and the check is redone with a new third point
                # on the next iteration.
                if collinearity_check(p1, p2, p3):
                    # Something subtle here but we can mutate
                    # `pruned_path` freely because the length
                    # of the list is check on every iteration.
                    pruned_path.remove(pruned_path[i+1])
                else:
                    i += 1
            return pruned_path
        
added or copied and pasted this part from the excercises of planning lesson 2!     


### Adjust your deadbands

            def local_position_callback(self):
                    if self.flight_state == States.TAKEOFF:
                        if -1.0 * self.local_position[2] > 0.95 * self.target_position[2]:
                            self.waypoint_transition()
                    elif self.flight_state == States.WAYPOINT:
                        if np.linalg.norm(self.target_position[0:2] - self.local_position[0:2]) < 5.0:   # deadband adjusted
                            if len(self.waypoints) > 0:
                                self.waypoint_transition()
                            else:
                                if np.linalg.norm(self.local_velocity[0:2]) < 1.0:
                                    self.landing_transition()

deadband adjusted to 5.0 m replacing 1.0 m


### Error with medial axis

            Traceback (most recent call last):
              File "/home/turtle/miniconda3/envs/fcnd/lib/python3.6/site-packages/udacidrone/drone.py", line 378, in notify_callbacks
                fn()
              File "motion_planning.py", line 71, in state_callback
                self.plan_path()
              File "motion_planning.py", line 204, in plan_path
                self.send_waypoints()
              File "motion_planning.py", line 115, in send_waypoints
                data = msgpack.dumps(self.waypoints)
              File "/home/turtle/miniconda3/envs/fcnd/lib/python3.6/site-packages/msgpack/__init__.py", line 47, in packb
                return Packer(**kwargs).pack(o)
              File "msgpack/_packer.pyx", line 284, in msgpack._packer.Packer.pack
              File "msgpack/_packer.pyx", line 290, in msgpack._packer.Packer.pack
              File "msgpack/_packer.pyx", line 287, in msgpack._packer.Packer.pack
              File "msgpack/_packer.pyx", line 263, in msgpack._packer.Packer._pack
              File "msgpack/_packer.pyx", line 263, in msgpack._packer.Packer._pack
              File "msgpack/_packer.pyx", line 281, in msgpack._packer.Packer._pack
            TypeError: can't serialize 0
            takeoff transition
            
I have tried with Medial axis but facing this error! Without that the code works fine!

### Error with voronoi graph search
   

### Execute the flight
#### 1. Does it work?
It works! But with medial axis and voronoi graph search, it does not! Can you help me correcting them! I have heavily modified the planning_utils.py so as to use up all 2d options! For a 3D implementation, I have to modify the grid, and many other things! But first, please help me running the medial axis and voronoi graph in 2d!

### Double check that you've met specifications for each of the [rubric](https://review.udacity.com/#!/rubrics/1534/view) points.
  
# Extra Challenges: Real World Planning

For an extra challenge, consider implementing some of the techniques described in the "Real World Planning" lesson. You could try implementing a vehicle model to take dynamic constraints into account, or implement a replanning method to invoke if you get off course or encounter unexpected obstacles.


