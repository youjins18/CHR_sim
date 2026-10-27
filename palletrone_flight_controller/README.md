# palletrone_flight_controller

Low-level base-pose controller. It consumes the base fields of `/chr/reference`
and publishes only `/chr/actuator/palletrone`.

The node contains:

1. world-frame position PID with gravity feedforward;
2. quaternion attitude PID with bounded integral compensation for the suspended
   arm's static gravity moment;
3. an optional, disabled-by-default torque disturbance observer;
4. X-configuration roll/pitch/yaw thrust allocation followed by tangent-axis
   horizontal-force tilt allocation and actuator saturation.

`/chr/diagnostics/flight` keeps only the final body force, while separating
nominal attitude torque, DOB-estimated disturbance torque and final torque. The
DOB estimate is logged even when compensation is disabled; enabling compensation
applies `-dob_gain * dob_torque` to the nominal torque. The diagnostics also
include thrust, servo commands, allocator health and motor PWM. The PWM
conversion uses the current T-Motor CINE77 KV977 calibration
`F = 4.045973e-5 PWM^2 - 0.048579 PWM + 1.0186`, including its 0.8 normalized
output limit (1100--1740 us effective range). All coefficients remain ROS
parameters so later propulsion calibration can replace them without code edits.

Rotor order and axes match `chr_description/mujoco/palletrone.xml`: (+x,+y),
(-x,+y), (-x,-y), (+x,-y), with radial tilt axes. The measured plant radius is
0.230 m and the controller estimate is 0.229 m.

The MuJoCo plant retains the measured values: 5.874 kg total dynamic mass,
0.230 m rotor radius and 0.018 reaction-torque coefficient. The controller uses
slightly perturbed estimates (5.90 kg, 0.229 m and 0.0181) to represent
sub-percent measurement and assembly uncertainty without modifying the plant.
Gains and inertia are simulation starting values, not flight-tested values. Keep
the DOB disabled until the nominal inertia and actuator signs are identified
from hardware.
