Build new package - in ~/ros2_ws/src:

```
ros2 pkg create --build-type ament_python <package_name>
```

Build your package - in ~/ros2_ws:

```
colcon build --symlink-install --packages-select <package_name>
```