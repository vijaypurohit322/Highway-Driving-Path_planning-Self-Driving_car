# Udacity-CarND-Path-Planning

## Path Planning Project
The goal of this project is to drive a car around a simulated highway including traffic. In all previous projects we just used to drive only our car around the track, but this was a bit different. The simulator tries to inact the actual highway situation and we just need to write our code in such a way that car drives better than we drive!!! As always there are a number of constraints-:

* The car must not exceed the speed limit of 50mph on the highway
* The car does not exceed a total acceleration of 10 m/s^2
* The car does not exceed a total jerk of 10 m/s^3
* The car does not have collisions with any of the cars while changing lanes or driving inside a lane
* The car stays in its lane, except for the time between changing lanes.
* If the car ahead is moving slow, then we should be able to change lanes.

## Output
The final output video can be observed [here](https://youtu.be/Uv2Z6rKvxaA).

## Rubric Points

1. *The Code compiles correctly.*- The code complied successfully. I used spline approach for smoothening and I have added the spline.h file in the folder.

2. *The car is able to drive at least 4.32 miles without incident.* - The car drove around the highway easily. I tested for 9.2 miles and the car drove without any incidents, please check the output video for referrence.

3. *The car drives according to the speed limit.*- As I mentioned in Point 2 the car drove without any incidents, and maintaining speed limit for one of the incidents. The car drives at a max speed of 49.5 if there is no slow moving traffic ahead of car, if the car encounters a slow moving traffic, it tries to change the lane. If lane change is successful, then well and good otherwise car will lower its speed and keep moving in the same lane.

4. *Max Acceleration and Jerk are not Exceeded.* - True, the acceleration and jerk were in prescribed limits. If there is slow moving traffic ahead the car's acceleration is reduced by 5mph and immediately from 49 to 18 as it will produce a lot of jerk. Same is the case with the acceleration, it accelerates smoothly by 5mph.

5. *Car does not have collisions.*- There were no collisions, there were one or two incidents where there were collisions while changing lanes, but proper code was then implemented which I will explain in the pipeline.

6. *The car stays in its lane, except for the time between changing lanes.*- The car stays inside the lane, we are just responsible for providing the lane number, the control section is handeled by the simulator.

7. *The car is able to change lanes*- Whenever there is a slow moving traffic the car tries to change the lane and it does it successfully if there are not cars nearby.

8. *There is a reflection on how to generate paths.* Described below.

## Model Documentation

The simulator sends a number of things like Car's location, velocity, yaw rate, speed, frenet coordinates and sensor fusion data.  I implemented this project using a spline function, which fits a line to given points in a fairly smooth function. The classroom talks about the fifth order jerk minimizing trajectory but I decided to use spline as it is a well made library and it will reduce our computations. After fitting a line, I then feed points along that line back to the simulator.

### Prediction

The data from the sensor fusion and simulator is used to generate the prediction of what the other vehicles are likely to do. The sensor fusion data gives us data about the other cars nearby and we predict what they are likely to do. We change our behaviour on the basis of the Prediction of other cars behaviour

### Behaviour Planning 


My algo for behaviour planning is as follows-:

1. The car checks 30 m ahead if there is any car present or not.
2. If the senses a car 30 m ahead and it is slow moving so it tries to change the lane.
3. It tries to check the left lane first, if it free and contains no vehicle ahead of 30 m and 10 m behind the car. Behind because while changing lanes collision can happen with the cars moving point to point parallel.
4. If left lane is not available then it checks to switch to right lane and check if it free and contains no vehicle ahead of 30 m and 10 m behind the car.
5. If no option for lane change is there then it stays in the current lane without colliding with the car ahead.


### Trajectory Generation (Lines 415 - 516)

Our car calucates the trajectory based on its own speed, the speed of surrounding cars, current lane, intended lane and past points. To make trajectory smoother, we add last two points. If there are no previous points then we calculate the previous points from the current yaw and current car coordinates. Also the 3 points are added in next 30, 60, 90 meters to trajecotry. All these points are shifted to car reference angle.


### Final Logic for Switching Lanes
```
// function that returns if the car in the front of us is too close. Here we do not check any car coming from rear.

too_close = CheckIfCarIsTooClose(sensor_fusion, lane, car_s, prev_size, false);

if(too_close)
{
	cout<<"Car is too close"<<endl;
	int left_lane = lane - 1;
	int right_lane = lane + 1;
	if(left_lane >=0)
	{
		cout<<"left lane is"<<left_lane<<endl;
		bool too_close_left = CheckIfCarIsTooClose(sensor_fusion, left_lane, car_s, prev_size,true);
		if(too_close_left)
		{
			cout<<"Left Car is too close"<<left_lane<<endl;
			if(right_lane <= 2)
			{
				bool too_close_right = CheckIfCarIsTooClose(sensor_fusion, right_lane, car_s, prev_size, true);
				if(too_close_right)
				{
					// keep lane
				}
				else
				{
          // switch to right lane
					lane = right_lane;
				}
			}
			else
			{
				bool too_close_left1 = CheckIfCarIsTooClose(sensor_fusion, left_lane, car_s, prev_size,true);
				if(too_close_left1)
				{
					//keep lane
				}
				else
				{
          // switch to left lane
					lane= left_lane;
				}
			}
		}
		else
		{
      // switch to left lane
			lane= left_lane;
		}
	}
	else
	{
		cout<<"Trying Right lane as left lane is not possible"<<endl;
				bool too_close_right = CheckIfCarIsTooClose(sensor_fusion, right_lane, car_s, prev_size, true);
				if(too_close_right)
				{
					// keep lane
				}
				else
				{
        // switch to right lane
					lane = right_lane;
				}
		
	}
```
