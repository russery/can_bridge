# can_bridge
This board enables a quasi-star topology CAN bus, with correct 60Ω impedance on each bus. Each of the four connectors can connect to a different CAN bus. The CAN bridge terminates each connected bus and bridges it to the other buses that are connected. It also bridges 5V power between all of the connectors, and powers itself off the 5V bus.

![CAN bridge board](http://i.imgur.com/OkmIAWQ.png?2 "CAN bridge board")

## project files
This board was designed in Altium CircuitStudio, as an opportunity to play around with CircuitStudio. I've also exported a .pdf schematic (can_bridge.PDF) for anyone who doesn't have CircuitStudio. Gerber files, a BOM, and other manufacturing files are under the 'outputs' folder.

I've also built the board at OSHPark, so it's [available to you there](https://oshpark.com/shared_projects/wRxKJq7h).

## background
In a normal CAN bus, there all of the nodes are connected on a single bus, which is terminated by a 120Ω resistor at each end.
![normal CAN bus](http://i.imgur.com/MDjxx2U.png "normal CAN bus")

This is inconvenient in many robotics applications where you may want a bunch of CAN devices that are located in many different parts of a robot where a "star" bus topology would be much easier than a linear bus topology terminated at two ends. Sometimes you can get away with just implementing a poorly-terminated star topology bus. This works fine as long as the bus is relatively "small" relative to the speed of operation, and as long as the operating environment is relatively electromagnetic "quiet". However, it's against the CAN spec, and can cause trouble in harsh electromagnetic environments or on relatively large buses.

That's where this project comes in. This board allows up to four separate CAN buses to be connected together and individually terminated, while sharing data between the buses.
![star topology CAN bus with bridge](http://i.imgur.com/2w7mv6w.png "star topology CAN bus with bridge")

## design details
The magic of this board is enabled by the [ON Semiconductor AMIS42770](http://datasheet.octopart.com/AMIS42770ICAW1RG-ON-Semiconductor-datasheet-8360231.pdf). This chip has dual CAN transcievers on it, and can bridge data between the two buses. It also has an external RX/TX interface that allows it to connect to another AMIS42270, allowing up to four buses to be bridged together by two chips. They're a little pricey, and a large-ish SOIC-20 package, but have some really cool features and work pretty well. 

This board connects to each CAN bus via a Molex Micro-fit 3.0 connector (PN: 0430450401, Mating connector: 0430250400). All four of the connectors are identical and interchangeable (it doesn't matter which one powers the board, or which bus plugs into each one).

The connectors carry the CAN bus, 5V power and ground. Note that 5V power must be provided on at least one connector in order for the board to work.

Here's the connector pinout:
<table>
<tr><td>*pin*</td><td>*purpose*</td></tr>
<tr><td>1</td><td>ground</td></tr>
<tr><td>2</td><td>+5V</td></tr>
<tr><td>3</td><td>CAN_L</td></tr>
<tr><td>4</td><td>CAN_H</td></tr>
</table>
![connector pinout](http://i.imgur.com/t5lMGmX.png "connector pinout")

## testing
I did some brief testing of the latency between CAN buses, as this is the primary performance criterion I was concerned with. The latency across buses is what will determine the maximum operating frequency of the CAN buses this can be used with.

Here is an example capture of data on the CAN-H line of two buses, with CAN-H-IN being the side connected to the CAN master, and CAN-H-OUT the side connected to the CAN slave. In this capture, I believe the slave was transmitting, as you can actually see that CAN-H-IN lags CAN-H-OUT ever so slightly:
![data on input and output](http://i.imgur.com/YNX6UgS.png "data on input and output")
I made this measurement and subsequent ones between buses 1 and 3, so as to capture the worst-case propagation delay (the signal must go through both of the transciever chips to be bridged between these two buses).

Here I've zoomed in on an edge, and measured the transition latency:
![latency measurement](http://i.imgur.com/09QWqca.png "latency measurement")

And here's a latency measurement on data transmission going the other direction (master to slave):
![latency measurement 2](http://i.imgur.com/y67Nuhp.png "latency measurement 2")

In each case, the latency is right about 120ns. This is on the order of the propagation delay seen on a 40 meter bus (1/C * 40m = 133nS). Referencing the timing table in the AMIS42770 datasheet, this is around what we would expect. Worst-case latency might be higher, but is likely still well within the propagation segment of a 1Mbps CAN bus (I believe 500ns is typical for this). 

The bus-to-bus delay is great enough that I would be concerned about daisychaining multiples of these bus bridges together, or operating them on long buses. In any case, it's probably worth verifying the can bridge's performance in your target system, with the cable lengths you plan on using. 

