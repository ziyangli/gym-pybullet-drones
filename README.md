# gym-pybullet-drones
[OpenAI Gym environment](https://gym.openai.com/envs/#classic_control) based on [PyBullet](https://github.com/bulletphysics/bullet3) for multi-agent reinforcement learning with quadrotors 

<img src="files/readme_images/helix.gif" alt="alt text" width="360"> <img src="files/readme_images/helix.png" alt="alt text" width="450">

- The default `DroneModel.CF2X` dynamics are based on [Bitcraze's Crazyflie 2.x nano-quadrotor](https://www.bitcraze.io/documentation/hardware/crazyflie_2_1/crazyflie_2_1-datasheet.pdf)

- Everything after a `$` is entered on a terminal, everything after `>>>` is passed to a Python interpreter

- Suggestions and corrections are very welcome in the form of [issues](https://github.com/JacopoPan/gym-pybullet-drones/issues) and [pull requests](https://github.com/JacopoPan/gym-pybullet-drones/pulls), respectively




## Overview

|                                   | `gym-pybullet-drones` | [AirSim](https://github.com/microsoft/AirSim) | [Flightmare](https://github.com/uzh-rpg/flightmare) |
|---------------------------------: | :-------------------: | :-------------------------------------------: | :-------------------------------------------------: |
|                         *Physics* | PyBullet              | FastPhysicsEngine/PhysX                       | *Ad hoc*/Gazebo                                     |
|                       *Rendering* | PyBullet              | Unreal Engine 4                               | Unity                                               |
|                        *Language* | Python                | C++/C#                                        | C++/Python                                          |  
|           *RGB/Depth/Segm. views* | **Yes**               | **Yes**                                       | **Yes**                                             |
|             *Multi-agent control* | **Yes**               | **Yes**                                       | **Yes**                                             |
|                   *ROS interface* | No                    | **Yes**                                       | **Yes**                                             |
|            *Hardware-In-The-Loop* | No                    | **Yes**                                       | No                                                  |
|         *Fully steppable physics* | **Yes**               | No                                            | **Yes**                                             |
|             *Aerodynamic effects* | Drag, downwash, ground| Drag                                          | Drag                                                |
|          *OpenAI [`Gym`](https://github.com/openai/gym/blob/master/gym/core.py) interface* | **Yes** | No | **Yes**                                             |
| *RLlib [`MultiAgentEnv`](https://github.com/ray-project/ray/blob/master/rllib/env/multi_agent_env.py) interface* | **Yes**  | No | No                           |




## Performance
Simulation **speed-up with respect to the wall-clock** when using
- *240Hz* (in simulation clock) PyBullet physics for **EACH** drone
- **AND** *48Hz* (in simulation clock) PID control of **EACH** drone
- **AND** nearby *obstacles* **AND** a mildly complex *background* (see .GIFs)
- **AND** *24FPS* (in sim. clock), *64x48 pixel* capture of *6 channels* (RGBA, depth, segm.) on **EACH** drone

|                                  | Lenovo P52 (i7-8850H/Quadro P2000) | 2020 MacBook Pro (i7-1068NG7) |
| -------------------------------: | :--------------------------------: | :---------------------------: |
| Rendering                        | OpenGL \*\*\*                      | CPU-based TinyRenderer        | 
| Single drone, **no** vision      | 15.5x                              | 16.8x                         |
| Single drone **with** vision     | 10.8x                              | 1.3x                          |
| Multi-drone (10), **no** vision  | 2.1x                               | 2.3x                          |
| Multi-drone (5) **with** vision  | 2.5x                               | 0.2x                          |

> \*\*\* **on Ubuntu only, uncomment the line after `self.CLIENT = p.connect(p.DIRECT)` in `BaseAviary.py`**

> While it is easy to—consciously or not—[cherry pick](https://en.wikipedia.org/wiki/Cherry_picking) statistics, \~5kHz PyBullet physics (CPU-only) is faster than [AirSim (1kHz)](https://arxiv.org/pdf/1705.05065.pdf) and more accurate than [Flightmare's 35kHz simple single quadcopter dynamics](https://arxiv.org/pdf/2009.00563.pdf)

> Multi-agent 6-ch. video capture at \~750kB/s with CPU rendering (`(64*48)*(4+4+2)*24*5*0.2`) is comparable to [Flightmare's 240 RGB frames/s](https://arxiv.org/pdf/2009.00563.pdf) (`(32*32)*3*240`)—although in more complex [Unity environments](https://arxiv.org/pdf/2009.00563.pdf)—and up to an order of magnitude faster on Ubuntu, with OpenGL rendering




## Requirements
The repo was written using *Python 3.7.6* on *macOS 10.15* and tested on *Ubuntu 18.04.05*

Major dependencies are [`gym`](https://gym.openai.com/docs/),  [`pybullet`](https://docs.google.com/document/d/10sXEhzFRSnvFcl3XxNGhnD4N2SedqwdAvK3dsihxVUA/edit#), 
[`stable-baselines3`](https://stable-baselines3.readthedocs.io/en/master/guide/quickstart.html), [`rllib`](https://docs.ray.io/en/master/rllib.html) and [`ffmpeg`](https://ffmpeg.org) (for video recording only)
```
$ pip install gym
$ pip install pybullet
$ pip install stable-baselines3
$ pip install 'ray[rllib]'
$ brew install ffmpeg                       # on macOS
$ sudo apt install ffmpeg                   # on Linux
```
Using a [`conda` environment](https://github.com/JacopoPan/a-minimalist-guide#install-conda), 
dependencies (except `ffmpeg`), can be installed from file
```
$ cd gym-pybullet-drones/
$ conda create -n myenv --file /files/conda_req_list.txt
```




## Installation
The repo is structured as a [Gym Environment](https://github.com/openai/gym/blob/master/docs/creating-environments.md)
and can be installed with `pip install --editable`
```
$ git clone https://github.com/JacopoPan/gym-pybullet-drones.git
$ cd gym-pybullet-drones/
$ pip install -e .
```




## Use
There are 2 basic scripts in `examples/`: `fly.py` and `learn.py`

- `fly.py` runs an independent flight **using PID control** implemented in class [`DSLPIDControl`](https://github.com/JacopoPan/gym-pybullet-drones/tree/master/gym_pybullet_drones/control/DSLPIDControl.py)
```
$ conda activate myenv                      # If using a conda environment
$ cd gym-pybullet-drones/examples/
$ python fly.py
```
> Tip: use the GUI's sliders and button `Use GUI RPM` to override the control with interactive inputs

<img src="files/readme_images/wp.gif" alt="alt text" width="360"> <img src="files/readme_images/wp.png" alt="alt text" width="450">

<img src="files/readme_images/crash.gif" alt="alt text" width="360"> <img src="files/readme_images/crash.png" alt="alt text" width="450">

- `learn.py` is an **RL example** to learn take-off using `stable-baselines3`'s [A2C](https://stable-baselines3.readthedocs.io/en/master/modules/a2c.html) or `rllib`'s [PPO](https://docs.ray.io/en/master/rllib-algorithms.html#ppo)
```
$ conda activate myenv                      # If using a conda environment
$ cd gym-pybullet-drones/examples/
$ python learn.py
```
<img src="files/readme_images/learn1.gif" alt="alt text" width="400"> <img src="files/readme_images/learn2.gif" alt="alt text" width="400">
<img src="files/readme_images/learn3.gif" alt="alt text" width="400"> <img src="files/readme_images/learn4.gif" alt="alt text" width="400">

Other scripts in folder `examples/` are:

- `compare.py` which replays and compare to a trace saved in [`example_trace.pkl`](https://github.com/JacopoPan/gym-pybullet-drones/tree/master/files/example_trace.pkl)
```
$ conda activate myenv                      # If using a conda environment
$ cd gym-pybullet-drones/examples/
$ python compare.py
```
<img src="files/readme_images/trace_comparison.gif" alt="alt text" width="360"> <img src="files/readme_images/trace_comparison.png" alt="alt text" width="450">

- `downwash.py` is a flight script with only 2 drones, to test the downwash model
```
$ conda activate myenv                      # If using a conda environment
$ cd gym-pybullet-drones/examples/
$ python downwash.py
```

<img src="files/readme_images/downwash.gif" alt="alt text" width="360"> <img src="files/readme_images/downwash.png" alt="alt text" width="450">

- `physics.py` is an accessory script that can be used to understand PyBullet's force and torque APIs for different [URDF links](http://wiki.ros.org/urdf/XML/link) in `p.WORLD_FRAME` and `p.LINK_FRAME`
```
$ conda activate myenv                      # If using a conda environment
$ cd gym-pybullet-drones/examples/
$ python physics.py
```
> Tip: also check the examples in [pybullet-examples](https://github.com/JacopoPan/pybullet-examples)

- `_dev.py` is a script continuously updated with the latest features of `gym-pybullet-drones` like RGB, depth and segmentation views from each drone's POV or compatibility with RLlibs's [`MultiAgentEnv`](https://docs.ray.io/en/latest/rllib-env.html#multi-agent-and-hierarchical) class
```
$ conda activate myenv                      # If using a conda environment
$ cd gym-pybullet-drones/examples/
$ python _dev.py
```

<img src="files/readme_images/rgb.gif" alt="alt text" width="270"> <img src="files/readme_images/dep.gif" alt="alt text" width="270"> <img src="files/readme_images/seg.gif" alt="alt text" width="270">




## BaseAviary
A flight arena for one (ore more) quadrotor can be created as a child class of `BaseAviary()`
```
>>> env = BaseAviary( 
>>>       drone_model=DroneModel.CF2X,      # See DroneModel Enum class for other quadcopter models 
>>>       num_drones=1,                     # Number of drones 
>>>       visibility_radius=np.inf,         # Distance at which drones are considered neighbors, only used for multiple drones 
>>>       initial_xyzs=None,                # Initial XYZ positions of the drones
>>>       initial_rpys=None,                # Initial roll, pitch, and yaw of the drones in radians 
>>>       physics: Physics=Physics.PYB,     # Choice of (PyBullet) physics implementation 
>>>       freq=240,                         # Stepping frequency of the simulation
>>>       aggregate_phy_steps=1,            # Number of physics updates within each call to BaseAviary.step()
>>>       gui=True,                         # Whether to display PyBullet's GUI, only use this for debbuging
>>>       record=False,                     # Whether to save a .mp4 video (if gui=True) or .png frames (if gui=False) in gym-pybullet-drones/files/, see script /files/ffmpeg_png2mp4.sh for encoding
>>>       obstacles=False)                  # Whether to add obstacles to the environment
````
And instantiated with `gym.make()`—see [`learn.py`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/examples/learn.py) for an example
```
>>> env = gym.make('rl-takeoff-aviary-v0')  # See learn.py
```
Then, the environment can be stepped with
```
>>> obs = env.reset()
>>> for _ in range(10*240):
>>>     obs, reward, done, info = env.step(env.action_space.sample())
>>>     env.render()
>>>     if done: obs = env.reset()
>>> env.close()
```




### Action Spaces
The action space's definition of an environment must be implemented in each child of [`BaseAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/BaseAviary.py) by function
```
>>> def _actionSpace(self):
>>>     ...
```
In [`CtrlAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/CtrlAviary.py) and [`VisionCtrlAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/VisionCtrlAviary.py), it is a [`Dict()`](https://github.com/openai/gym/blob/master/gym/spaces/dict.py) of [`Box(4,)`](https://github.com/openai/gym/blob/master/gym/spaces/box.py) containing the drones' commanded RPMs

The dictionary's keys are `"0"`, `"1"`, .., `"n"`—where `n` is the number of drones

The action space of [`MARLFlockAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/MARLFlockAviary.py) has the same structure but values are normalized in range `[-1, 1]`

The action space of [`RLTakeoffAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/RLTakeoffAviary.py) is a single [`Box(4,)`](https://github.com/openai/gym/blob/master/gym/spaces/box.py) normalized to the `[-1, 1]` range

Each child of [`BaseAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/BaseAviary.py) also needs to implement a preprocessing step translating actions into RPMs
```
>>> def _preprocessAction(self, action):
>>>     ...
```
[`CtrlAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/CtrlAviary.py), [`VisionCtrlAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/VisionCtrlAviary.py), [`MARLFlockAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/MARLFlockAviary.py), and [`RLTakeoffAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/RLTakeoffAviary.py) all simply clip the inputs to `MAX_RPM`



### Observation Spaces
The observation space's definition of an environment must be implemented by every child of [`BaseAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/BaseAviary.py)
```
>>> def _observationSpace(self):
>>>     ...
```
In [`CtrlAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/CtrlAviary.py), it is a [`Dict()`](https://github.com/openai/gym/blob/master/gym/spaces/dict.py) of pairs `{"state": Box(20,), "neighbors": MultiBinary(num_drones)}`

The dictionary's keys are `"0"`, `"1"`, .., `"n"`—where `n` is the number of drones

Each [`Box(20,)`](https://github.com/openai/gym/blob/master/gym/spaces/box.py) contains the drone's
- X, Y, Z position in `WORLD_FRAME` (3 value, meters unless normalized)
- Quaternion orientation in `WORLD_FRAME` (4 values)
- Roll, pitch and yaw angles in `WORLD_FRAME` (3 values, radians unless normalized)
- The velocity vector in `WORLD_FRAME` (3 values, m/s unless normalized)
- Angular velocities in `WORLD_FRAME` (3 values, rad/s unless normalized)
- Motors' speeds (4 values, RPM)

Each [`MultiBinary(num_drones)`](https://github.com/openai/gym/blob/master/gym/spaces/multi_binary.py) contains the drone's own row of the multi-robot system [adjacency matrix](https://en.wikipedia.org/wiki/Adjacency_matrix)

The observation space of [`MARLFlockAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/MARLFlockAviary.py) has the same structure but normalized to the `[-1, 1]` range

The observation space of [`RLTakeoffAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/RLTakeoffAviary.py) is a single [`Box(20,)`](https://github.com/openai/gym/blob/master/gym/spaces/box.py), normalized to the `[-1, 1]` range

The observation space of [`VisionCtrlAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/VisionCtrlAviary.py) is the same as[`CtrlAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/CtrlAviary.py) but also includes keys `rgb`, `dep`, and `seg` (in each drone's dictionary) for the matrices containing the drone's RGB, depth, and segmentation views

To fill/customize the content of each `obs`, every child of [`BaseAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/BaseAviary.py) needs to implement
```
>>> def _computeObs(self, action):
>>>     ...
```




### Reward, done, and info
`Reward`, `done` and `info` are computed by the implementation of 3 functions in every child of [`BaseAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/BaseAviary.py)
```
>>> def _computeReward(self, obs):
>>>     ...                                 # float or dict of floats
>>> def _computeDone(self, obs):
>>>     ...                                 # bool or dict of bools
>>> def _computeInfo(self, obs):
>>>     ...                                 # dict or dict of dicts
```
See [`RLTakeoffAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/RLTakeoffAviary.py) and [`MARLFlockAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/MARLFlockAviary.py) for example implementations




### Drag, Ground Effect, and Downwash Models
Simple drag, ground effect, and downwash models can be included in the simulation initializing `BaseAviary()` with `physics=Physics.PYB_GND_DRAG_DW`, these are based on the system identification of [Forster (2015)](http://mikehamer.info/assets/papers/Crazyflie%20Modelling.pdf) (Eq. 4.2), the analytical model used as a baseline for comparison by [Shi et al. (2019)](https://arxiv.org/pdf/1811.08027.pdf) (Eq. 15), and [DSL](https://www.dynsyslab.org/vision-news/)'s experimental work

Check the implementations of `_drag()`, `_groundEffect()`, and `_downwash()` in [`BaseAviary`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/envs/BaseAviary.py) for more detail




## Control
Folder [`control`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/control/) contains the implementations of 2 PID controllers

[`DSLPIDControl`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/control/DSLPIDControl.py) (for `DroneModel.CF2X/P`) and [`SimplePIDControl`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/control/SimplePIDControl.py) (for `DroneModel.HB`) can be used as
```   
>>> ctrl = [DSLPIDControl(env) for i in range(num_drones)]                          # Initialize "num_drones" controllers
>>> ...
>>> for i in range(num_drones):                                                     # Compute control for each drone
>>>     action[str(i)], _, _ = ctrl[i].computeControlFromState(.                    # Write the action in a dictionary
>>>                                    control_timestep=env.TIMESTEP,
>>>                                    state=obs[str(i)]["state"],
>>>                                    target_pos=TARGET_POS)
```




## Logger
Class [`Logger`](https://github.com/JacopoPan/gym-pybullet-drones/blob/master/gym_pybullet_drones/utils/Logger.py) contains helper functions to save and plot simulation data, as in this example
```
>>> logger = Logger(logging_freq_hz=freq, num_drones=num_drones)                    # Initialize the logger
>>> ...
>>> for i in range(NUM_DRONES):             # Log information for each drone
>>>     logger.log(drone=i,
>>>                timestamp=K/env.SIM_FREQ,
>>>                state= obs[str(i)]["state"],
>>>                control=np.hstack([ TARGET_POS, np.zeros(9) ]))   
>>> ...
>>> logger.save()                           # Save data to file
>>> logger.plot()                           # Plot data
```




## Citation
If you wish, please refer to this work as
```
@MISC{gym-pybullet-drones2020,
    author = {Panerati, Jacopo and Zheng, Hehui and Zhou, SiQi and Xu, James and Prorok, Amanda and Sch\"{o}llig, Angela P.},
    title = {Learning to Fly: a PyBullet-based Gym environment to simulate and learn the control of multiple nano-quadcopters},
    howpublished = {\url{https://github.com/JacopoPan/gym-pybullet-drones}},
    year = {2020}
}
```




## References
- Nathan Michael, Daniel Mellinger, Quentin Lindsey, Vijay Kumar (2010) [*The GRASP Multiple Micro UAV Testbed*](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.169.1687&rep=rep1&type=pdf)
- Benoit Landry (2014) [*Planning and Control for Quadrotor Flight through Cluttered Environments*](http://groups.csail.mit.edu/robotics-center/public_papers/Landry15)
- Julian Forster (2015) [*System Identification of the Crazyflie 2.0 Nano Quadrocopter*](http://mikehamer.info/assets/papers/Crazyflie%20Modelling.pdf)
- Carlos Luis and Jeroome Le Ny (2016) [*Design of a Trajectory Tracking Controller for a Nanoquadcopter*](https://arxiv.org/pdf/1608.05786.pdf)
- Shital Shah, Debadeepta Dey, Chris Lovett, and Ashish Kapoor (2017) [*AirSim: High-Fidelity Visual and Physical Simulation for Autonomous Vehicles*](https://arxiv.org/pdf/1705.05065.pdf)
- Guanya Shi, Xichen Shi, Michael O’Connell, Rose Yu, Kamyar Azizzadenesheli, Animashree Anandkumar, Yisong Yue, and Soon-Jo Chung (2019)
[*Neural Lander: Stable Drone Landing Control Using Learned Dynamics*](https://arxiv.org/pdf/1811.08027.pdf)
- Yunlong Song, Selim Naji, Elia Kaufmann, Antonio Loquercio, and Davide Scaramuzza (2020) [*Flightmare: A Flexible Quadrotor Simulator*](https://arxiv.org/pdf/2009.00563.pdf)

-----

> University of Toronto's [Dynamic Systems Lab](https://github.com/utiasDSL) / [Vector Institute](https://github.com/VectorInstitute) / University of Cambridge's [Prorok Lab](https://github.com/proroklab) / [Mitacs](https://www.mitacs.ca/en/projects/multi-agent-reinforcement-learning-decentralized-uavugv-cooperative-exploration)

