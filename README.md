# Rotary-Inverted-Pendulum
Detailed documentation of the modeling work for a QUBE-Servo rotary inverted pendulum (Furuta pendulum). The current objective is to describe the dynamics near the upright position, so that a balancing controller can be designed later. The pendulum is initially placed upright by hand. Swing-up from the downward position is a later stage.
Reading order
Document	What it explains
Notation and assumptions	The two angles, physical parameters, axes, units, and inertia definitions
Detailed derivation	Geometry, velocities, energies, Euler-Lagrange differentiation, nonlinear equations, linearization, and state-space equations
Numerical substitution	Parameter sources, derived constants, motor-voltage conversion, and every nontrivial matrix coefficient
The derivation explains both what is calculated and why it is needed. Intermediate algebra is included so that the center-of-mass translation terms, rotational terms, coupling signs, and motor back-EMF contribution can be followed without skipping steps.
Approach
Identify the independent coordinates: arm rotation theta and pendulum tilt alpha.
Locate the pendulum center of mass using geometry.
Differentiate position to find center-of-mass velocity.
Find kinetic energy of the arm and the pendulum, including translation and rotation where appropriate.
Find gravitational potential energy and form the Lagrangian.
Apply Euler-Lagrange once for each coordinate to obtain two nonlinear motion equations.
Linearize about the upright, stationary equilibrium.
Introduce four states and isolate the two angular accelerations by scalar substitution.
Include the DC motor equation so that the input is voltage.
Collect the coefficients into A, B, C, and D.
State and input convention
$$
x=\begin{bmatrix}\theta&\alpha&\dot\theta&\dot\alpha\end{bmatrix}^{T},\qquad u=V_m,\qquad y=\begin{bmatrix}\theta&\alpha\end{bmatrix}^{T}.
$$
Alpha is zero upright. At theta = 0, the arm points along +X and positive alpha tilts the pendulum toward -Y. The hinge moves in a horizontal circle around the motor axis. Angles are expressed in radians.
Completed mathematical work
Geometry and angle conventions.
Translational and rotational kinetic energies.
Gravitational potential energy and Lagrangian.
Nonlinear equations of motion with viscous damping.
Local linearization about upright.
Torque-input and voltage-input state equations.
Symbolic and numerical upright A, B, C, D matrices.
Scope and model assumptions
The numerical example uses QUBE-Servo 3 manual parameters and the damping values in the course brief. Links are treated as uniform slender rods. Pendulum axial inertia, separate rotor/hub inertia, and motor inductance are neglected in the baseline model. These are modeling choices, not measured properties or experimental validation.
The course brief uses j_p for pendulum inertia about its center of mass. To keep the axis explicit, this documentation uses J_c for that inertia and J_p for hinge inertia. They are related by the parallel-axis theorem; they must not be interchanged.
No claim is made that the pendulum has been balanced. A zero-input upright model is unstable for a small nonzero initial disturbance. With exactly zero initial state and zero input, the ideal mathematical model remains at equilibrium.
References
Course brief: Fall 2026 - Control Systems Lab Project, ELE 443, Parts 1 and 2. Supplies the modeling objectives, upright equilibrium, voltage input, angle outputs, and damping coefficients. The original handout is not redistributed here.
Quanser QUBE-Servo 3 User Manual, Table 2.2, manufacturer-authored PDF hosted by a distributor.
Quanser: Rotary Pendulum Control Challenge.
