# Feature Requests


## Category: Smarter Control Options


### Feedforward Torque Estimation
Apply predictive control to improve responsiveness beyond traditional feedback loops.  
This would allow the controller to anticipate torque demands based on command changes and known system dynamics, reducing lag.


### Gain Scheduling
Automatically adjust control gains based on operating conditions (e.g., velocity, load).  
Dynamic tuning can improve stability and performance across the full operating range.


### Motion Profiling
Smooth acceleration and deceleration using trapezoidal or S-curve profiles to reduce jerk.  
Beneficial for mechanical longevity and operator comfort.


### Filtering Techniques
Apply low-pass filters or complementary filters to smooth sensor noise and current measurements.  
This can improve control accuracy and reduce oscillations from noisy data.


### Z-Input from Incremental Encoder
Integrate Z-index pulse handling into encoder mode to establish absolute mechanical position at boot.  
This eliminates the need for a full rotation at startup, enabling faster and more reliable position referencing.