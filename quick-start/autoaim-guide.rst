自瞄使用说明
=============

1. 使用建议
-------------

* 自瞄3m内命中率高

* 视觉识别到一个敌人越久，预测就越准

* 自身处在左右急速运动或急停，或者敌人左右急停时自瞄可能会甩飞(原因:敌人状态从移动到急停的一瞬间，卡尔曼将会重新初始化以保证自身模型的稳定性)

* 使用自瞄打击小陀螺时先锁住看1~2s再发弹，看的时间越久自瞄预测就越准

* 对于在电容支持下高速陀螺或者移动的目标，可以等待几秒待其电容耗尽再打击(对于转速过快的目标将自己锁定在车辆装甲板中心)

* 如果自瞄失效，或者目标常规移动命中率都极低，使用手瞄，将问题反馈给视觉


2. 关于调试
-------------
.. note::

   有时视觉自瞄组成员会不在实验室或者出现问题看似是视觉BUG实则不然的情况，所以我们希望电控同学能掌握一些基本的调试方法，用以解决或者正确定位问题，能够独立进行调试，从而提高效率。有时就是几个指令，观察一下可视化的事情，根本不会涉及到视觉代码相关的层面，也可避免出现人等人的现象。

   所以也建议电控同学教会视觉同学对于遥控器或者客户端控制机器人的操作，以方便电控同学不在视觉也能对自瞄在车上进行调试。


2.1. 关于自瞄的启动
~~~~~~~~~~~~~~~~~~~

现在每台车自瞄都有自启动，如果上电后没有自瞄，``CTRL + ALT + T`` 打开终端输入以下指令

.. code-block:: bash

   #docker attach rv_runtime_autoStart</del

   cd ${workspace} # workspace改为ros2代码存放根目录
   bash rv.start.sh 

你将在终端看到串口的DEBUG信息，是不断刷新的绿色消息则自瞄启动正常

2.2. 关于foxglove可视化
~~~~~~~~~~~~~~~~~~~~~~~

在调试阶段视觉更多的操作是在开启foxglove下进行的，其可以图形可视化的显示出自瞄信息，在遇到自瞄甩飞、不识别装甲板、查看电控目标解算位置，或者需要参数调整等问题时打开终端输入以下指令

.. code-block:: bash

   
   #docker start rv_runtime_fox
   #docker attach rv_runtime_fox
   #有可能存在rv_runtime_fox不存在的情况，因为目前视觉关于这个的命名并未统一
   #可以使用以下指令查看具体名字
   #docker ps -a
   
   #在项目文件夹内内打开终端
   source install/setup.bash
   ros2 launch foxglove_bridge foxglove_bridge_launch.xml 


打开foxglove端后你能看到类似以下内容

  .. image:: image/1.png
     :width: 600 px


其中``serial/receive``为电控发给视觉的imu以及识别所需参数

``armor_solver/cmd_gimbal``为armor_solver节点对识别到二维坐标处理完成后发给电控的云台绝对角度

``tracker/target`` 是视觉滤波结果,调试时具有重要参考意义

``三维`` 这个窗口是对自己和目标的可视化建模，两个坐标系分别是 ``odom`` 坐标系(在最下面)， ``相机`` 坐标系(在上面)，四块装甲板是目标装甲板位置。视觉发送的所有坐标都在"odom"坐标系下。


.. note::
    此时晃动云台，三维窗口下显示的四块装甲板的位置应不发生改变，即其位置是在odom坐标系下定义的，应不随相机坐标系的变化而变化

2.3. 关于打击静态目标，距离不同打高打低的问题
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
修改pc上的home目录下的 ``/{workspace}/src/rm_bringup/config/launch_params.yaml`` 与 ``/ros_ws/src/rm_gimbal_description/urdf
/rm_gimbal.urdf.xacro`` 路径内的"rpy"部分为 “0 0 0” 如下图

  .. image:: image/2.png
     :width: 600 px

  .. image:: image/3.png
     :width: 600 px

然后修改串口包(rm_serial_driver)中的infantry_protocol.cpp,将其中的tmp_pitch改为固定值0，
后进入robomaster选手客户端使得云台pitch固定到绝对0点，
后用其余工具测量枪管上的pitch角，如果不为0则将其测得的误差放进rpy中的第二个参数，
以此迭代出正确的参数。


2024.4.10 Shakima first commit
2024.9.1 123456dfg changed



.. contents:: Table of Contents
   :depth: 2
   :local:
   
