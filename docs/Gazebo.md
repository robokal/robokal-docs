# Gazebo simulator

Gazebo is a physics simualtor which make it easy to examine robots algorithms.

## Building new world

In oreder to build a world or a model in the gazebo, we should create a sdf file.

A world sdf file should look as:

> xml version

> sdf version
>> world 
>>> model 


&nbsp;

An example to world file:

```
<?xml version='1.0'?>
<sdf version="1.5">
    <world name="default">
        <model name="ground_plane">
            <static>true</static>
            <link name="link">
              <collision name="collision">
                <geometry>
                  <plane>
                    <normal>0 0 1</normal>
                    <size>100 100</size>
                  </plane>
                </geometry>
                <surface>
                  <contact>
                     <collide_bitmask>0xffff</collide_bitmask>
                  </contact>
                  <friction>
                    <ode>
                      <mu>0.8</mu>
                      <mu2>0</mu2>
                    </ode>
                  </friction>
                </surface>
              </collision>
              <visual name="visual">
                <cast_shadows>false</cast_shadows>
                <geometry>
                  <plane>
                    <normal>0 0 1</normal>
                    <size>100 100</size>
                  </plane>
                </geometry>
                <material>
                  <script>
                    <uri>file://media/materials/scripts/gazebo.material</uri>
                    <name>Gazebo/Grey</name>
                  </script>
                </material>
              </visual>
            </link>
        </model>
        <model name ='box'>
          <pose>1 2 0 0 0 0</pose>
          <link name ='link'>
            <pose>0 0 .5 0 0 0</pose>
            <collision name ='collision'>
              <geometry>
                <box><size>1 1 1</size></box>
              </geometry>
            </collision>
            <visual name ='visual'>
              <geometry>
                <box><size>1 1 1</size></box>
              </geometry>
            </visual>
          </link>
        </model>
    </world>
</sdf>
```
&nbsp;

If you want to add a color to your model, you should add material to the visual tag:

```
<material>
  <script>
    <uri>file://media/materials/scripts/gazebo.material<uri>
    <name>Gazebo/Green</name>
  </script>
</material>
```