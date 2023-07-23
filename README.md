# AIVEHICLEGAME
 Vehicle Path following with traffic

Vehicle Path following with Traffic System for Games


NOTES


UE Version used - UE4.25 Later switched to UE4.27


1. Task - Draw a road track using spline


Approach - I first tried using the assets that come along with the advanced vehicle project in ue4 to build together a track. The grid isn’t subdivided so I tried to use instanced meshing while correcting for gaps by incorporating mesh length in the function I created for laying down track. Separately spawned the curved parts at the ends. The track was successfully created but the gap was low and this model was not flexible for different meshes. So, then I created my own Track Grid in blender, subdivided it appropriately, applied materials and imported into ue. The subdivided mesh allowed me to use spline mesh components and scale it using the set start and end function. The result was a flexible track that can be used with any form of spline.


Implementation notes - Created a Track class in C++. Inherited this class in Blueprints. This tack has two spline components for two loops. Created these splines in the editor by setting appropriate parameters. Then I created the Construction implementation for C++ class using spline mesh components. Also created accessor functions for spline components. 


There was a glitch when curved instances at the end of track A were disappearing upon close examination when looking towards the -x axis. I fixed this problem by adding a spline point in the middle of the curve.


While building the Win64 package for the project, the generated track was not appearing in the game. The reason for this is that the OnConstruction function doesn’t inform us about an error I was making, that is attaching a static spline mesh component to a movable rootcomponent. Somehow shifting logic to BeginPlay does inform us about this error. The solution was not to attach the generated spline mesh components to the root component as they will be static finally and move the logic to BeginPlay as the OnConstruction function somehow doesn’t make it into the final build. 


2. Task - Vehicle AI


Approach - I tried to look for AI functionality in the engine for vehicle classes. Since, as per my knowledge, I used the simple steering behaviors to find a tangent at the closest point on the curve and set a target at a position closest to a distance of look ahead in the direction of our tangent. This simple algorithm works well in this case. Used Mapping Range to map rotation values to steer the vehicle. Also set the throttle on the vehicle. Tested parameters manually till I found a sweet spot for the mapping range for rotation and value for throttle.


Implementation notes -  Modified the vehicle pawn class in C++, to create an AI path following function which is called every frame in Tick. Vehicle keeps pointers to the tracks in the level and also a pointer to a spline component which is the path it is following. We can switch the values pointed by the pointer to switch paths to follow. Also since there is only one track in the level I got its reference directly from the world instance.


3. Task - Traffic Lights


Approach - I thought of ways to make traffic lights in such a way to solve the track switching problem for the vehicle more generally in case of more than one track. I soon realized that it would be an overkill approach. Also I initially thought of store intersection data in the traffic lights in the form of an index for each track the intersection is at and then modify the AI algorithm for the vehicle to use that data by querying traffic lights. Another Idea was to split track in 4 segments and have traffic light return one of the segments based on direction of vehicle. After a bit of thought, I realized that the problem is symmetric for the vehicle and it can itself decide which way to go. This simplified the Traffic lights to implement their own functions. One of the problems for the vehicle waiting on Red light is not to waste time polling the traffic light to check the change of signal. It is the traffic light’s job to notify the vehicle waiting when the signal changes.


Implementation notes - 


Created a Traffic Light class in C++. It uses a signal enumeration to store light signals. It has a static mesh for Traffic Light and a box component to detect overlap with the vehicle, so the vehicle can know that it has reached an intersection. Vehicle can query traffic lights for the current signal/light. Has a delegate to notify the waiting vehicle when signal changes in case it has to wait at a red signal. Traffic Light works as a simple state machine, so there are state functions for each of the signals. Inherited in Blueprint to change mesh upon change of state to animate the traffic light. Traffic Light starts Red and changes to Green and then yellow at intermediate intervals of 10 seconds. When the Traffic light changes to yellow or green it notifies the vehicle of the fact. 


The vehicle upon overlapping with the traffic light queries it and handles the case for signal accordingly. In case of a red light, the vehicle binds its signal switch handling function to the traffic light’s delegate so that it can be notified when the signal changes. In the case of yellow light we accelerate as if to catch the light. We then handle the task of randomly choosing one of 3 directions ahead and moving. 


Added Traffic lights in the Map area and adjusted box component.  


4. Task - Randomly choose one of 3 ways ahead


Approach - This was a somewhat tricky part of the problem. In addition to the above problems, one of the solutions was to store two splines for handling left and right turns at intersections to guide the vehicle out of the intersection. At the end, the solution I adopted was to let the vehicle do the task autonomously. 


Implementation - We switch paths on the track at intersections in case we turn right or left. This logic simplifies much of the problem. Then I modified the following AI path to calculate the dot product of tangent and actor’s forward direction. We use a minimum value for the dot product to check against. If the value is less then we consider the tangent at an intersection turn and then we need to handle the case for moving the vehicle left or right, otherwise it would turn in the same direction every time for a particular intersection. We set a boolean variable in the intersection handling function for turning left and another variable for turning right. We set the appropriate variable and if the variable is set, we turn in its direction (right or left of the vehicle) until the dot product with the tangent doesn’t become more than the minimum value.


5. Task - Adding points on drifting at the curve


Approach - Use a box component and on the end of overlap add points to the vehicle's score.


Implementation - Created a class in C++ with a box component for giving out drift points. Adjusting it in Level.


6. Task - Fuel property


Approach - Vehicle burns fuel based on the throttle. When the fuel drops to zero, the vehicle stops.


Implementation - Burn the fuel in Tick based on a consumption rate. Check if the fuel has depleted to zero and if this is the case, stop the vehicle, otherwise allow the AI path following algorithm to run.


7. Task - Add UMG Widget Status Bar to the Game


Approach/Implementation - Used Blueprints and added the widget to viewport in Level Blueprints. Accessed Fuel, Speed and Score from the vehicle. Had to get used to the UMG framework. Added border for black background in widget, added a horizontal box as its child and then added slots in horizontal box using spacers to fill in the gap. With correct offsets for Text blocks and progress bar, the widget was set up correctly. Tested the widget.


8. Task - Creating a UE4 Generic Character.


Approach - The vehicle pawn has a Physx vehicle movement component attached to it, which requires a controller to be attached to the pawn for it to apply its movement. As we change the default pawn, we need to make sure to give an AI Controller to the vehicle pawn. Alternatively, Chaos vehicle movement component lets you set a flag which disables the requirement for a controller for movement input but this is an experimental plugin as of UE4.27 and is recommended not to be used in products to be shipped.


Implementation - Added Third Person Character Content Pack to Project. Placed a Third Person Character Pawn in the level. Changed Game Mode Settings to set the character as default pawn. Set the auto possess property for character pawn to player 0.  Created a new AI Controller class for the vehicle pawn. Set the auto possess property on vehicle pawn to disabled and set it to auto possess the AI controller placed in the world. Placed an instance of an AI controller in the world.
