# Dynamic Collaborative Motion Planning in a Factory Workspace

Motion planning and execution for a KUKA LBR iiwa 14 cobot in static and dynamic environments, built with ROS Noetic, MoveIt, RViz and Gazebo.

The project compares intrinsically optimized sampling-based planners (RRT*, PRM*) against non-optimal planners (RRT, PRM) post-processed with shortcutting, and uses a closed-loop, time-sliced planning/execution strategy so the arm can react to obstacles that appear mid-motion — a workaround for industrial controllers that don't support interrupting a running trajectory.

This was a group project for Project 3-1 at Maastricht University, developed together with Tristan Dormans, Tomasz Mizera, Alex Andreescu, Nishan Mistry and Maarten Kooij. Original team repository: [Arezim/Project-3-1](https://github.com/Arezim/Project-3-1).

## Repository layout

```
motion-planning-1/          Main workspace (Docker + ROS Noetic + MoveIt)
  docker-compose.yml        ROS Noetic desktop-full container with X11 GUI support
  up.sh / shell.sh / ...    Helper scripts to start the container and open shells
  ws/src/
    iiwa_description/       KUKA iiwa 14 URDF and meshes
    iiwa_moveit_config/     MoveIt configuration (planning groups, OMPL settings)
    iiwa_dynamic_demo/      Demo scripts and benchmarks
      scripts/              Motion demos: A-to-B, static/dynamic obstacles, replanning
      benchmarks/           Planner benchmark results (CSV) and analysis script

motion-planning-2/          Second workspace: custom robot model via MoveIt Setup Assistant
  src/configuration/        MoveIt config (OMPL, CHOMP, STOMP, Pilz pipelines)
  src/my_robot_description/ URDF and Gazebo spawn launch
```

## Running it

Requires Docker and an X11 server (native on Linux; WSLg or VcXsrv on Windows).

```bash
cd motion-planning-1
./up.sh        # prepares X11 auth and starts the ros-noetic container
./shell.sh     # opens a shell inside the container
```

Inside the container, build the workspace and launch MoveIt + RViz:

```bash
cd /root/ws
catkin init && catkin config --extend /opt/ros/noetic
catkin build
source devel/setup.bash
roslaunch iiwa_moveit_config demo.launch
```

Then, in a second container shell, run the demos:

```bash
source /root/ws/devel/setup.bash
rosrun iiwa_dynamic_demo a_to_b_demo.py              # basic A-to-B motion
rosrun iiwa_dynamic_demo a_to_b_with_obstacle.py     # plan around a static obstacle
rosrun iiwa_dynamic_demo a_to_b_dynamic_obstacle.py  # obstacle appears mid-motion
rosrun iiwa_dynamic_demo dynamic_replan_demo.py      # time-sliced execution + replanning
```

Step-by-step instructions and troubleshooting: [iiwa_demo_instructions.txt](motion-planning-1/ws/iiwa_demo_instructions.txt).

Demo videos: [environment + motion planning](https://youtu.be/OaXe6YaLlrA), [obstacle avoidance](https://www.youtube.com/watch?v=SShL30cCFh4).

> Note: `motion-planning-2` expects the `iiwa_description` package in its `src/` as well — copy it over from `motion-planning-1/ws/src/` before building.

## Benchmarks

`iiwa_dynamic_demo/scripts/parallel_benchmark*.py` run the planner comparison (RRT, PRM, RRT*, PRM*, with and without shortcutting) in static and dynamic scenes; results land in `iiwa_dynamic_demo/benchmarks/` as CSV and are summarized by `analyze_benchmarks.py`. In our experiments, non-optimal planners plus shortcutting slightly outperformed the intrinsically optimized planners in planning time and trajectory complexity.

## Hardware setup

The physical experiments used a KUKA LBR iiwa 14 with an OnRobot RG6 gripper and an Intel RealSense D435 depth camera, driven from a lab machine running Ubuntu 20.04. The simulation stack in this repo reproduces that environment in Docker so it runs on any machine.
