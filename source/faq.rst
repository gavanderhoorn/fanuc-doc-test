Troubleshooting
===============


Overview
--------

This page lists some of the possible errors and issues encountered when using the `fanuc_driver`_ package. The *Remedy* subsections provide possible solutions.

If you encounter an error not listed on this page or the provided remedy does not seem to work, please contact the developers using the `ROS-Industrial <https://groups.google.com/forum/?fromgroups#!forum/swri-ros-pkg-dev>`_ Google group (direct mail: `ROS-Industrial <mailto:swri-ros-pkg-dev@googlegroups.com>`_).

.. note::

   Make sure to check the `Known Issues <http://wiki.ros.org/fanuc/indigo/known_issues>`_ page before sending a message to the mailing list.


Controller
----------

KAREL programs are invisible on the Program Select window
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After copying the KAREL programs onto your controller, you do not see them on the *Program Select* window, nor can you change the displayed program ``TYPE`` to ``KAREL Progs``.

**Cause**: Listing and accessing KAREL programs is not enabled on the controller.

**Remedy**: Enable the listing of KAREL programs by setting the ``$KAREL_ENB`` system variable to ``1``. Open the *SYSTEM Variables* screen (``Menu → NEXT → SYSTEM → Variables``), and scroll down to ``$KAREL_ENB``. Press ``ENTER``, and change the value to ``1``. The modification is immediate, you do not need to restart the controller.


MEMO-159: Convert failed in PROG
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

While trying to copy a new or updated version of any of the ROS-Industrial KAREL programs, the TP shows the following error:

.. code-block:: none

  Duplicate creation TYPE mismatch
  MEMO-159 Convert failed in PROG
  VARS-014 Create type - KL_TYPE failed

Where ``PROG`` is any KAREL program, and ``KL_TYPE`` is a KAREL type used in ``PROG``.

**Cause**: The type or structure of a variable used in ``PROG`` has changed in the updated version, and the KAREL runtime system cannot convert the variable to the new type. This error can occur even after removing ``PROG.PC`` file from the controller, as the related ``.vr`` file (or *variable record*, the file that stores information on variables) is not automatically deleted upon deleting the p-code file.

**Remedy**: Remove the ``PROG.VR`` file from the controller. This can be done from the *Program Select* window, as well as by using the ``Menu → FILE`` menu (switch to the ``MD:`` device). If the copy attempt left a ``PROG.PC`` file, remove it as well. The copy operation should now succeed.

In some cases, other (ROS-Industrial) KAREL programs or libraries have references to the old type as well and these will have to be removed too. Always first delete the ``.PC`` file, then the ``.VR``.

**Note**: Removing the ``.VR`` file will result in loss of the values of the variables stored in the ``.VR``. Backup important data before deleting the files. The ``PROG.VA`` file should provide you with an ASCII rendering of the ``.VR`` file.


TPIF-013: Other program is running
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Upon trying to (re)start the ``ros`` TP program, the TP shows the following error:

.. code-block:: none

  TPIF-013 Other program is running

**Cause**: This is caused by another TPE or KAREL user program that is still running on the controller. You cannot start new user programs while others are still running.

**Remedy**: Make sure no other user programs are running or paused on the controller before trying to start the ROS programs. ``Fctn → ABORT ALL`` can be used to abort any running programs.

**Note**: You may need to use ``ABORT ALL`` twice for programs that have network sockets that are in the ``ACCEPT`` state. For the ``ros_relay`` and ``ros_state`` programs, this is when the TP shows the following on the User Menu:

.. code-block:: none

  12345 I RSTA Waiting for ROS state prox
  12345 I RREL Waiting for ROS traj relay

and the ROS PC is not connected to the controller. Select ``Fctn → ABORT ALL`` twice, then try again.


KAREL driver
------------

HOST-144: Comm Tag error
^^^^^^^^^^^^^^^^^^^^^^^^

The TP shows the following on the User Menu:

.. code-block:: none

  12345 E PROG cfg err, TAG idx: N

Where ``PROG`` is any of ``RSTA`` or ``RREL``, and ``N`` is a Server Tag index number.

**Cause**: the configured Server Tag does either not exist, is not correctly configured, or is not in the correct state.

**Remedy**: make sure you have configured the Tag correctly, and that the configuration of the respective ROS-Industrial program uses the correct tag number. Refer to the `Server Tags <http://wiki.ros.org/fanuc/Tutorials/hydro/Configuration#Server_Tags>`_ section of the installation tutorial for information on Tag setup.


FILE-032: Illegal parameter
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The TP shows the following on the User Menu:

.. code-block:: none

  12345 E PROG cfg error: -1
  12346 E PROG check cfg

Where ``PROG`` is any of ``RSTA`` or ``RREL``.

**Cause:** the configuration of the respective program is either uninitialised, or has not been completed.

**Remedy:** make sure the configuration of all ROS-Industrial programs has been completed, and that the ``checked`` entry is set to ``TRUE``. Refer to the `KAREL Programs <http://wiki.ros.org/fanuc/Tutorials/hydro/Configuration#KAREL_Programs>`_ section of the installation tutorial for information on the setup of the KAREL and TPE programs.


INTP-320: (LIBSSOCK, N) Undefined built-in
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nothing is shown on the User Menu, and all ROS-Industrial programs (``ros_state``, ``ros_relay``) have been aborted.

**Cause**: this is most likely caused by attempting to start the programs on a controller that does not have all required options installed. For ``libssock``, this is most likely option R648 (*User Socket Messaging*) (resulting in the missing ``MSG_*`` built-ins).

**Remedy**: make sure the controller has all required options installed (and activated). Refer to the `Prerequisites <http://wiki.ros.org/fanuc/Tutorials/hydro/Installation#Prerequisites>`_ section of the installation tutorial for information on which options are required.

If the controller has the required options installed, please report the issue to the mailinglist and / or the `ros-industrial/fanuc <https://github.com/ros-industrial/fanuc/issues>`_ repository. Do not forget to include any relevant information (ROS release, version of `fanuc_driver`_, controller type and software and used manipulator).


jnt_pkt_srlise err: ERRNO
^^^^^^^^^^^^^^^^^^^^^^^^^

The TP shows the following on the User Menu:

.. code-block:: none

  12345 E PROG jnt_pkt_srlise err: ERRNO
  12345 I PROG Waiting for ROS ..

Where ``PROG`` is any of ``RSTA`` or ``RREL``, and ``ERRNO`` is a negative error number (possible values: ``-67213``).

**Cause**: The ROS client disconnected the TCP socket connection at the moment the KAREL program tried to write to or read from it.

**Remedy**: This is not an error. The state proxy and trajectory relay will handle the disconnect and wait for a new connection.


exec_move err: -1
^^^^^^^^^^^^^^^^^

The TP shows the following on the User Menu:

.. code-block:: none

  12345 E RREL exec_move err: -1
  12345 I RREL Waiting for ROS ..

The ROS console shows:

.. code-block:: none

  [ERROR] [1234567890.123456789]: Socket sendBytes failed, rc: -1. Error: 'Broken pipe' (errno: 32)
  [ERROR] [1234567890.123456789]: Failed to connect to server, rc: -1. Error: 'Transport endpoint is already connected' (errno: 106)

  (the same error repeated)

  [ERROR] [1234567890.123456789]: Failed to connect to server, rc: -1. Error: 'Transport endpoint is already connected' (errno: 106)
  [ERROR] [1234567890.123456789]: Timeout connecting to robot controller.  Send new motion command to retry.

Sending a new motion command does not always work.

**Cause**: The ROS client asked the trajectory relay to move to a point in the current trajectory that the robot is unable to reach (violation of joint limits). The relay aborts execution of the trajectory and disconnects the ROS client in that case.

**Remedy**: Verify that the joint limits configured on the Fanuc controller correspond to those in the used urdf (in the support package). The MoveIt motion planners depend on these limits to generate valid trajectories. Update the limits in the xacro macro if there are any differences. Be sure to regenerate the urdf and any MoveIt packages that depend on the it.

**Note**: In some cases it may be necessary to restart the ``ros`` TPE program on the controller.


ROS
---

RViz only shows 'collision quality' models
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After starting RViz (either by using the ``test_...launch`` files in the support packages, or when running any of the MoveIt ``planning_execution`` launch files), the visualisation only shows collision detail models, even when *Visual Enabled* is selected.

**Cause:** the support packages only include the collision quality models, as we do not currently have permission to distribute the detailed CAD models.

**Remedy:** none. A possible work-around would involve re-exporting the support package meshes from SolidWorks or any other suitable program. A complicating factor is that correct display of the model depends on proper origins of the meshes, which are not necessarily identical to those of the original models. Additionally, users would need access to the original models.


Interactive marker in RViz cannot be dragged around
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After starting ``demo.launch`` or ``moveit_planning_execution.launch`` from any of the Fanuc MoveIt configurations, the 6D interactive marker cannot be dragged from the starting position, and the terminal window shows the following error:

.. code-block:: none

  [ERROR] [1234567890.123456789]: Exception caught while handling end-effector update: ikfast exception: /path/to/ikfast/plugin/solver.cpp:LINE_NR: polyroots8: Assertion 'rawcoeffs[0] != 0' failed

**Cause:** the manipulator is most likely a singular configuration, for which the IK plugin cannot find any valid solutions. ``demo.launch`` (and ``moveit_planning_execution.launch`` without the ``sim:=false`` argument) does not use real joint state data, but a simple joint space interpolator, for which the initial state puts all joints at their 'zero position'. For some models this is a singular configuration.

**Remedy:** if possible, use a different initial state, command the manipulator to move away (in joint space) from the singular configuration, or (if in simulation) use the *random valid* option for the *Goal State* and click *Plan and Execute* button. As soon as the manipulator is out of the singular configuration, dragging the interactive marker should start working again.


Robot stops at seemingly random points during trajectory execution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The TP shows the following on the User Menu:

.. code-block:: none

  12345 I RREL Trajectory stop

The ROS console shows:

.. code-block:: none

  [ERROR] [1234567890.123456789]: Controller is taking too long to execute trajectory (the expected upper bound for the trajectory execution was X.Y seconds). Stopping trajectory.
  [ INFO] [1234567890.123456789]: MoveitSimpleControllerManager: Cancelling execution for
  [ INFO] [1234567890.123456789]: Execution completed: TIMED_OUT

Where ``X.Y`` is some duration in seconds.

**Cause:** the MoveIt *Trajectory Execution Manager* detects a trajectory execution overrun, and aborts execution of the trajectory. This is often due to the physical robot not being able to attain the speeds specified in the trajectory. The Fanuc KAREL trajectory relay overrides specified motion speed, resulting in the quoted error message.

**Remedy:** increase the MoveIt duration scaling parameter, or disable duration monitoring completely. See the *The Trajectory Execution Manager* section on the `Executing Trajectories with MoveIt! <http://wiki.ros.org/fanuc/Troubleshooting/MoveIt%20Trajectory%20Execution%20Manager>`_ page on the MoveIt wiki or `How do I disable execution_duration_monitoring ? <http://answers.ros.org/question/196586/how-do-i-disable-execution_duration_monitoring/>`_ on ROS Answers for more information.


Failed to load byte array
^^^^^^^^^^^^^^^^^^^^^^^^^

The TP shows the following on the User Menu:

.. code-block:: none

  12345 I RSTA Connected
  12345 I RREL Connected

The ROS console shows:

.. code-block:: none

  [ERROR] [1234567890.123456789]: Set buffer size: 1060, larger than MAX:, 1024
  [ERROR] [1234567890.123456789]: Failed to load byte array

**Cause:** one possible cause is a difference in endianness between the robot and the PC running ROS. The Fanuc stack includes preconfigured launch files taking the endianness of the specific robot controller into account. If the actual controller uses a different endianness, the communication will not work properly.

**Remedy:** try to provide the launchfile with a different value for the ``use_bswap`` argument. For most controllers, this is set to ``true``. If appending ``use_bswap:=false`` solves the problem, please report the issue to the mailinglist and / or the `ros-industrial/fanuc <https://github.com/ros-industrial/fanuc/issues>`_ repository. Do not forget to include any relevant information (ROS release, version of `fanuc_driver`_, controller type and software and used manipulator).


Unable to reconnect to the robot
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The TP shows the following on the User Menu:

.. code-block:: none

  12345 I RSTA Waiting for ROS state prox
  12345 I RREL Waiting for ROS traj relay
  12345 I RSTA Connected

or:

.. code-block:: none

  12345 I RREL Connected

The ROS console shows:

.. code-block:: none

  [ERROR] [1234567890.123456789]: Failed to connect to server, rc: -1. Error: 'Connection timed out' (errno: 110)
  [..]
  [ERROR] [1234567890.123456789]: Timeout connecting to robot controller.  Send new motion command to retry.

**Cause:** this error can be caused by an incomplete and / or unexpected shutdown of the ROS-Industrial nodes on the PC, leaving the ROS-Industrial programs on the controller in an inconsistent state. In such a case, the socket on the controller is still waiting for a timeout to occur and blocks any new connection attempt in the meantime.

**Remedy:** either wait for the timeout on the controller to occur, or forcibly stop the ROS-Industrial programs on the controller using ``Fctn → ABORT (ALL)``. Now restart the programs on the controller, then restart the ROS nodes on the PC.


.. Links

.. _fanuc_driver: http://wiki.ros.org/fanuc_driver
