---
layout: post
author: Hemachander Rubeshkumar
tags: [portfolio, in-progress, simulink, wind-energy]
permalink: /turbine-simulink/
title: CWC Turbine Simulink
---
## Why Simulate?
From my time in the Comet Wind Team and a short semester working on OpenFAST, I have been tasked with wind turbine control systems a few times, and a common wall with trying out or testing those controls is time. Last competition year, there wasnt too much time to really test out every variable, algorithm, or tuning constant in the tunnel. Building a turbine is a huge effort, even at the smaller 1-2 meter scale of CWC, and wind tunnel testing time is scarce. Its costly, it takes a lot of lead time to get it scheduled, and even then its almost never enough time. 

## The Model
To work around this limitation, I started working on a Simulink Model to model the turbine so we can test different control algorithms while its being worked on. It takes the simulated wind, convert it into torque from this "wind hitting the blades," take that torque into a generator block, and take out the rpm, which affects the torque calculation... etc. This loop is what forms the basis of the model

![Info.json](/images/Simulink_Model.png)
<div style="text-align: center;">
    Simulink Model of the Turbine
</div>

First, the wind is modelled as it would be in the competition, as a stairstep of integer wind speeds with some variation. To accomplish this, I used a code block to generate the ramping integer wind speeds from 5-13, and added a brown noise block to mimic more realistic conditions.

This wind speed then gets processed by the Aerodynamics Block. To explain how it works, I need to explain some basic principles of wind turbines. The TSR, or Tip-Speed-Ratio, is the ratio between the tangential velocity at the very end of the blades, and the current wind speed. This is important because different blades optimize for different TSRs and it determines power efficiency of the blades, or Cp. To do this, the model approximates the curve with a guassian distribution, being at peak performance of Cp = .5 at TSR = 4. Our model takes in the wind speed and current rpm to find the current TSR, finds the relevant point on the distribution, and gets a Cp value. This Cp value is then plugged into the Torque equation. T = 0.5 Ro(air resistivity) * A (Area of the circle that the blades travel) * v^3 (wind velocity) * Cp. 

That torque value then gets sent to the PMSM block, which can model any Permanent Magnet Stator Motor. For our case, we use a BLDC or Brushless DC Motor, which is a particular kind of PMSM. Essentially, the motor outputs a voltage for a given torque, and that voltage gets sent through the modelled electrical components. The resistor is the most importand factor, as it can affect the rpm of the motor by providing more or less resistance to the wind. Counter-intuitively, higher resistance does not necessarily mean harder to turn, in fact its quite the opposite. Think of a pinwheel on a lawn. It doesn't have any electrical opposition to spinning, and it doesnt have a complete electrical loop, essentially an infinite resisor. A low resistance closed loop would act like an inductor, opposing the change in current, and in our case, the rotation of the turbine blades.

To model the ability to code custom control algorithms onto the resistance, a code block was conncted to a variable resistor block. Once the wires are connected, and the circuitry in-place, we can see the simulation results.

![Info.json](/images/Simulink.png)
<div style="text-align: center;">
    A complete plot from a Simulink Turbine simulation run
</div>

## Further considerations
This model still seems to have some odd behaviors that need to be debugged
The approximation in the Aerodynamics block can be improved
There are many parameters in the PMSM block that need to be explored
The competition has current limits and other electrical criteria that need to be checked 
The pitching of the blades is not modelled yet (used for Phase 3 of power curve)

# NOTE that this project is an active work-in-progress
