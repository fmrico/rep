REP: XXX
Title: Motion Capture Systems
Author: Francisco Martín Rico
Status: Active
Type: Draft
Type: Standards Track
Content-Type: text/x-rst
Created: 01-Apr-2024
Post-History: 


Abstract
========

This REP proposes an standard the use of Motion Capture Systems in ROS 2.

It provides ROS 2 interface types that a Motion Capture Systems driver 
produces, reference frames in which this data is produced, as well as 
any aspect related to the use of Motion Capture Systems in ROS 2.


Motivation
==========

Motion Capture Systems (mocap systems, in short) are very important in 
Robotics. They are critical when testing localization algorithms and commanding 
groups of indoor drones or AR applications, among many other applications.

Typically, most proprietary control software programs calibrate,
configure, and manage the information obtained from their cameras. Most vendors
provide libraries for different applications with an SDK for connecting from any OS
and receiving position data. ROS 2 mocap drivers use these SDKs to provide the mocap
systems information to the nodes that require it. ROS 2 already has many packages available 
with drivers for mocap systems.

This REP defines the standards that should guide the development of the mocap system 
drivers so that any user application can be developed independently of the mocap 
system used, benefiting the reusability of applications and the use of this type 
of systems in ROS 2.


Specification
=============


Interfaces
----------

mocap_msgs/msg/Marker.msg
'''''''''''''''''''''''''

Most mocap systems use markers as the minimum detection unit, which are easily detectable 
elements in space, typically a small reflective ball.

::

  # This message contains the information for a single mocap marker
 
  # Time of mocap data acquisition, and the coordinate frame ID
  std_msgs/Header header

  int8 USE_NAME=0
  int8 USE_INDEX=1
  int8 USE_BOTH=2

  int8 id_type                     # A marker can be identified by a name, a integer, or both
  int32 marker_index               # The marker id as an integer (id_type = USE_INDEX|USE_BOTH) 
  string marker_name               # The marker id as a string (id_type = USE_NAME|USE_BOTH)       
  geometry_msgs/Point translation  # The marker position


mocap_msgs/msg/MarkerArray.msg
''''''''''''''''''''''''''''''

Markers are not published one by one but in a message that contains all the markers in a 
detection, numbered with a consecutive counter that increments in every detection.

::

  # This message contains the information for an array of mocap markers
 
  # Time of mocap data acquisition, and the coordinate frame ID
  std_msgs/Header header

  uint32 seq                       # A continous detection number
  mocap_msgs/Marker[] markers      # The array of markers


mocap_msgs/msg/RigidBody.msg
''''''''''''''''''''''''''''

Most mocap systems can detect rigid bodies, which allows us to know their orientation in 
space, in addition to their orientation. This message contains the array of markers, which
always follow the same order. 


::

  # This message contains the information for a single mocap rigid body
 
  # Time of mocap data acquisition, and the coordinate frame ID
  std_msgs/Header header

  string rigid_body_name           # An id of the rigid body assigned by the mocap system
  mocap_msgs/Marker[] markers      # The ordered array of markers
  geometry_msgs/Pose pose          # The Pose (positon and oriantation) of the center of the
                                   # rigid body


mocap_msgs/msg/RigidBodyArray.msg
'''''''''''''''''''''''''''''''''

All the rigid bodies detected in the same frame can be published in the same frame number.

::

  # This message contains the information for an array of rigid bodies
 
  # Time of mocap data acquisition, and the coordinate frame ID
  std_msgs/Header header

  uint32 seq                              # A continous detection number
  mocap4r2_msgs/RigidBody[] rigid_bodies  # The array of rigid bodies detected


Coordinate Frames
-----------------

The recommended frame identifier is ``mocap``. Mocap system drivers can (optionally but recommended, set as a boolean parameter ``publish_tf``) publish 
the TF that connects frames related to the mocap system with the other existing frames.

* If the mocap system detects the position of an isolated robot, the ``mocap`` frame will be the parent frame of ``odom``. Mocap system drivers can optionally publish the TF that connects both frames.

.. raw:: html

  <div class="mermaid">
  graph TD
    mo[mocap]
    od[odom]
    bf[base_footprint]
    mo --> od
    od --> bf
  </div>

* If the mocap system detects the position of a robot localized on a map, the ``mocap`` frame will be the parent frame of ``map``.

.. raw:: html

  <div class="mermaid">
  graph TD
    mo[mocap]
    ma[map]
    od[odom]
    bf[base_footprint]
    mo --> ma
    ma --> od
    od --> bf
  </div>

* If more than one mocap system coexists simultaneously, there will be a parent frame ``mocap`` whose children are each mocap system and the other frames. For example:

.. raw:: html

  <div class="mermaid">
  graph TD
    mo[mocap]
    moa[mocap_A]
    mob[mocap_B]
    moc[mocap_C]
    ma[map]
    od[odom]
    bf[base_footprint]
    mo --> moa
    mo --> mob
    mo --> ma
    ma --> od
    od --> bf
  </div>


Complementary specifications
----------------------------

* All computers involved in mocap systems, including those that run the vendor's software, should be synchronized using ``ntp`` or any other more precise mechanism.
* It is recommended that LifeCycle Nodes be used to implement the mocap system drivers to activate/deactivate the publication of mocap data.

Rationale
=========

* **Redundant headers in ``mocap_msgs/msg/MarkerArray.msg`` and ``mocap_msgs/msg/RigidBodyArray.msg``**: Timestamps of headers in ``*Array.msg`` messages can be different to their contents (markers or rigid bodies) to differentiate the capture time and the publication time.
* **Frames of multiple mocap systems**: It is possible to use more than one mocap system, and we should avoid repeating the same frames (for example, ``map`` or ``base_footprint``) in different branches in the same TF tree. To relate the coordinate positions of the detections of each frame, one global ``mocap`` frame should be chosen (it can match one of them). In this case, the TF publication that connects each mocap system with the ``mocap`` frame should be mandatory, and the ``frame_id`` of the messages should be of the specific mocap system that produced the detection.

Reference Implementation
========================

To be provided

Terminology
===========

Some terms used in this document, which will be described in much more detail in the specification:

- **ROS 2 Interface** or **Interface Type** - a ROS 2 message, service, or action.
- **MOCAP System** - Motion Caption System in short. This term refers to the complete system: hardware (cameras, hubs,..), the vendor propietary software and the ROS 2 mocap driver.
- **MOCAP System driver** - It refers to the nodes that access to the mocap system detections (usually though an SDK) and publish the data as ROS 2 interfaces.
- **Type Description** - A data structure representing a parsed type source, which will be equal regardless of source format such as ``.msg`` or ``.idl`` if the described type is the same.


References
==========

Copyright
=========

This document has been placed in the public domain.



..
   Local Variables:
   mode: indented-text
   indent-tabs-mode: nil
   sentence-end-double-space: t
   fill-column: 70
   coding: utf-8
   End:
