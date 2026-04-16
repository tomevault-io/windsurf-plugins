---
trigger: always_on
description: - Prefer modern c++ or python features like class, oop.
---

# Project general coding guidelines

## Code Style
- Prefer modern c++ or python features like class, oop.
- Use consistent indentation (4 spaces)
- Limit line length to 120 characters
- Use spaces around operators and after commas
- Place opening braces on the same line as the control statement
- Use single quotes for strings in Python and double quotes in C++

## Project Structure
- follow ros2 package structure for robotics projects
- Organize files into logical directories (e.g., src/, include/, tests/, docs/)

## Error Handling
- Use exceptions for error handling in Python and C++
- Catch specific exceptions rather than using a generic catch-all
- Clean up resources (e.g., close files, release memory) in error cases

## abstraction and Modularity
- Design code with modularity in mind, breaking down functionality into smaller, reusable components
- Use interfaces or abstract classes to define contracts for components
- Encapsulate implementation details to reduce dependencies between modules
- Follow the Single Responsibility Principle, ensuring each module or class has one reason to change  

## best Practices
- Write code that is easy to read and understand
- Avoid premature optimization; focus on clarity and correctness first
- Use design patterns where appropriate to solve common problems
- Regularly review and refactor code to improve quality and maintainability 

## Ros2 Specific Guidelines
- Follow ROS 2 coding standards and conventions
- Use ROS 2 logging mechanisms for debug and error messages
- Log error messages with sufficient context for debugging
- Use ROS 2 parameter server for configurable settings
- Leverage ROS 2 launch files for starting nodes and managing configurations
- Utilize ROS 2 message types and services appropriately
- Ensure compatibility with ROS 2 middleware (e.g., DDS)
- Implement proper shutdown procedures for ROS 2 nodes
- Handle ROS 2 callbacks efficiently to avoid blocking the main thread
- Use ROS 2 timers for periodic tasks
- Follow ROS 2 security best practices when handling sensitive data
- Integrate with ROS 2 tools for simulation and visualization (e.g., RViz, Gazebo)
- Ensure proper use of ROS 2 lifecycle management for nodes
- Follow ROS 2 community guidelines for contributing to open-source projects
- Use ROS 2 services and actions for request-response and long-running tasks
- Implement proper message filtering and throttling to manage data flow
- Use ROS 2 bag files for data recording and playback during testing
- Ensure compatibility with different ROS 2 distributions
- Follow ROS 2 package structure and naming conventions
- Utilize ROS 2 tools for building, testing, and deploying packages
- Ensure proper use of ROS 2 communication mechanisms (topics, services, actions)
- Adhere to ROS 2 package structure and naming conventions

## ROS2 Python Specific Guidelines
- Use rospy for ROS 2 Python nodes
- Follow Python-specific best practices in ROS 2 context
- Use rospy logging functions for debug and error messages
- Leverage rospy parameters for configurable settings
- Utilize rospy timers for periodic tasks
- Implement proper shutdown procedures for rospy nodes
- Handle rospy callbacks efficiently to avoid blocking the main thread
- Use rospy services and actions for request-response and long-running tasks

## moveit2 Specific Guidelines
- Follow MoveIt 2 coding standards and conventions
- Use MoveIt 2 planning interfaces for motion planning tasks
- Leverage MoveIt 2 collision checking mechanisms
- Utilize MoveIt 2 kinematics solvers appropriately
- Ensure compatibility with MoveIt 2 plugins and extensions
- Implement proper error handling for MoveIt 2 operations
- Follow MoveIt 2 community guidelines for contributing to open-source projects
## Moveit2 task constructor Specific Guidelines 
- Use MTC for complex task planning and execution
- Define tasks using MTC's task and stage abstractions
- Utilize MTC's built-in stages for common operations (e.g., MoveTo, Pick, Place)
- Implement custom stages when necessary for specialized behavior
- Use MTC's error handling and recovery mechanisms
# AI Coding Assistant Notes — arctos (focused)

This file gives concise, actionable guidance for AI coding agents working in this repository. Keep answers specific, reference files, and prefer minimal, testable changes.

**Architecture Overview**
- **ROS2 multi-package workspace:** key packages include [arctos_bringup](arctos_bringup), [arctos_description](arctos_description), [arctos_hardware_interface](arctos_hardware_interface), [arctos_moveit_config](arctos_moveit_config), [arctos_interfaces](arctos_interfaces).
- **Hardware & controllers:** the bridge between robot and ROS is in [arctos_hardware_interface/src/arctos_hardware_interface.cpp](arctos_hardware_interface/src/arctos_hardware_interface.cpp) and controller YAMLs in [arctos_bringup/config/ros2_controllers.yaml](arctos_bringup/config/ros2_controllers.yaml) and [arctos_bringup/config/real_controllers.yaml](arctos_bringup/config/real_controllers.yaml).
- **Simulation integration:** Gazebo and MoveIt are wired via launch files in [arctos_bringup/launch](arctos_bringup/launch) and [arctos_description/launch](arctos_description/launch). Use these to reproduce runtime behaviors.

**How components communicate**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/roachhuang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
