# raspiController_Server
#！请使用 python3 runOnRapberryPi.py (或 python3 NewVersion.py ) 命令打开文件，使用python2打开将不能正常工作。

（NewVersion.py中对控制电机的线程进行了合并，未经测试，注意，未经测试

建议与特制安卓端一起使用https://github.com/icecoins/runOnAndroid

在使用这个文件之前，请把树莓派与控制电机的引脚按照文件中的定义连接，

或是在自行决定且连接完后更改文件中的引脚定义。

请确保树莓派与使用控制端的安卓手机处于同一wifi下，

且确保树莓派上socket连接、输出视频流的端口没有被阻塞。

请自行修改绑定的端口，与安卓手机上将使用的端口保持一致即可。

本项目中的默认使用了mjpg-streamer产生视频流，在启动本文件时上述软件会被调用。

若需要自定义视频流参数，请自行修改代码。

若使用其他软件，请自行修改代码。

可以在ssh中对手机端发送信息，将显示在控制页面右上角，三行一清，

此功能存在于sendThread中，请自行修改以符合使用者的发送要求。

本文件有维护连接功能，请不要直接ctrl z退出，否则被监听的端口将不会立即释放。

如需退出，请输入q、Q、w、W中的一个并回车，本文件将很快关闭。

注意：本文件有非常多的bug，可能打开就崩溃，请不要过于惊讶。

Version 25.10.2021
