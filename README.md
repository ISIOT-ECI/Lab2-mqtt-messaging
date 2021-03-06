
### Software Engineering for IoT and BigData

### Lab 2 - IoT devices communication

### Exercise to be done in groups of two - Pick your partner!

### Required elements.

1. Elements from [Lab-01](https://github.com/ISIOT-ECI/Lab1-Sensors)

### Lab Goal

In this exercise, you and your partner will build two separate IoT devices (microcontroller + electronic circuit) that communicate with each other. Together, these two devices will simulate a scenario where Device A is in a factory, controlling a manufacturing process, continuously measuring and transmitting the environment parameters of the process. Device B (placed out of the factory), on the other hand, will receive the aforementioned environment parameters, and based on their values, it will trigger visual alarms to alert a human operator. The human operator, once the visual alarms are triggered, should be able to decide to stop the process being carried by Device A by pressing a button on the Device B's circuit. The human operator should be able to resume the process when he decides it is convenient, therefore, the environment variables should continue to be transmitted regardless the process is running or stopped.


### Device A - Monitoring device in the factory:
#### Hardware configuration
- A ESP8266 microcontroller, with a photoresistor and a humidity/temperature sensor (DHT11) circuits connected to it, as in [Lab-01](https://github.com/ISIOT-ECI/Lab1-Sensors).
- To represent the manufacturing process state, a LED will be also included (ON = process in action, OFF = process stopped)

#### Microcontroller Software
- The manufacturing process should be ON by default.
- The microcontroller must publish, every 2 seconds, the light and temperature readings in a topic on the MQTT broker.
- The requests for stopping or resuming the manufacturing process will be posted as a message on another topic on the MQTT broker. Therefore, the microcontroller should be subscribed to such topic, and continuously check for new messages. In this prototype, the effect of receiving a STOP or RESUME message is simply turning OFF and ON the LED.


### Device B - Remote monitoring and control:
#### Hardware configuration
- A ESP8266 microcontroller, with two LEDs, one as an alarm for Temperature and the other as an alarm for Ligh intensity in the environment.
- A push-button circuit, as in [Lab-01](), attached to one of the microcontroller's digital pins.

#### Microcontroller Software
- The microcontroller should be subscribed to the topic (or topics) where temperature and light readings will be published by Device A. When such values are above a given threshold, the corresponding LED should be turned on. Likewise, the LEDs should be turned off once the values are below their corresponding thresholds.
- Pressing the push button should post a STOP or RESUME message (depending on the status of the process) on the corresponding topic. In this case, you should consider using hardware interrupts, as in Lab-01, to detect when the button was pushed.



## Additional considerations

1. It is recommended to use the official micropython's MQTT client libraries: [umqtt.robust](https://github.com/micropython/micropython-lib/tree/master/umqtt.robust). *Note: we didn't use the umqtt.robust2 package due to problems with the ESP8266, but you are free to try it.*

	You can install these libraries throug [upip (micro pip)](https://docs.micropython.org/en/latest/reference/packages.html), by opening a REPL prompt (as in Lab-00) and entering:

	```python
	import upip
	upip.install('micropython-umqtt.robust')
	```

	You can check the [documentation of this library](https://pypi.org/project/micropython-umqtt.simple/) or the [umqtt.simple source code](https://github.com/micropython/micropython-lib/blob/master/umqtt.simple/umqtt/simple.py) (which is fairly simple and mostly documented) to see the parameters of the *publish*, *subscribe*, *set_callback*, *wait_msg* and *check_msg* methods. Pay special attention to the difference between *wait_msg* and *check_msg*. 

2. We suggest to use a free plan of [CloudAMQP](https://www.cloudamqp.com/) as the MQTT broker. Once you have configured your broker, you can check in the instance details its HOST (Load Balanced), USER & VHOST, and PASSWORD. When using this broker with umqtt, take the following into consideration:
	- To make a connection to the broker, the user string is the user given by CloudAMQP, but twice, sepparated by a colon. E.g. if the user is *exegpeeg*, the user is given to the mqtt client as follows:

		```python
		sub = MQTTClient('unique-identifier','thehost.com',1883,'exegpeeg:exegpeeg','ThePassword')
		```
	- The topics must have the above username as a prefix. For example, for the above user to subscribe or to post data on the broker's topic '/feeds/data', it must use 'exegpeeg:exegpeeg/feeds/data'. For example:

		```python
		client.publish('exegpeeg:exegpeeg/feeds/data', '33')
		```

3. In this laboratory one of the devices needs to perform two tasks simultaneously: posting sensor readings, and checking messages from a topic. The ESP8266 doesn't support threads, but you can instead use [the Micropython's asynchronous I/O scheduler](https://docs.micropython.org/en/latest/library/uasyncio.html). For example, the following allows the microcontroller to keep control of two independent tasks: keep blinking a red LED for a second, and keep blinking a blue LED for 100ms. Note that using *await uasyncio.sleep* instead of *time.sleep* ensures that the CPU won't be suspended, and instead it will yield the control to the other taks (i.e., taskB will be processed while taskA is in 'sleep' mode).


	```python
	async def taskA():
   		 while True:       
   		 	#turn ON the RED LED
        		await uasyncio.sleep_ms(1000)
        		#turn OFF the RED LED
			await uasyncio.sleep_ms(1000)

	async def taskB():
   		while True:
			#turn ON the BLUE LED	
        		await uasyncio.sleep_ms(100)
			#turn OFF the BLUE LED	
			await uasyncio.sleep_ms(100)
			
	loop = uasyncio.get_event_loop()
	loop.create_task(taskA())
	loop.create_task(taskB())
	loop.run_forever()

	```



4. Use the [microcontroller's unique ID](https://docs.micropython.org/en/latest/library/machine.html) as MQTT client's unique identifier. 


### Deliverables (to be submitted on moodle)

- Source code and pictures of both devices (breadboard + NodeMCU)
