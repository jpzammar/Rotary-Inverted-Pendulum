# Notation, geometry, and modeling assumptions

## Points and axes

| Symbol | Meaning |
| --- | --- |
| O | Point on the motor axis at the height of the pendulum hinge |
| P | Pendulum hinge at the end of the rotating arm |
| G | Pendulum center of mass |
| X, Y | Fixed horizontal Cartesian axes |
| Z | Fixed vertical axis, positive upward |

The hinge is the joint that allows the pendulum to swing relative to the arm. It is distinct from the motor axis. Its distance from the motor axis is r. As the arm rotates, P travels in a circle parallel to the QUBE top surface, while the pendulum swings in a vertical plane perpendicular to the arm.

## Coordinates and states

| Symbol | Meaning | Unit |
| --- | --- | --- |
| theta | Arm rotation, positive counterclockwise viewed from above | rad |
| alpha | Pendulum tilt measured from the upward vertical | rad |
| theta_dot | Arm angular velocity | rad/s |
| alpha_dot | Pendulum angular velocity relative to its hinge | rad/s |
| theta_ddot | Arm angular acceleration | rad/s^2 |
| alpha_ddot | Pendulum tilt angular acceleration | rad/s^2 |
| x1 | theta | rad |
| x2 | alpha | rad |
| x3 | theta_dot | rad/s |
| x4 | alpha_dot | rad/s |

Theta = 0 is a chosen arm reference, not a unique gravitational equilibrium heading. Alpha = 0 is upright; alpha = pi is downward. This repository derives the local model at upright only.

The direction of positive alpha is fixed by geometry: at theta = 0, positive alpha tilts toward -Y. Consequently, the horizontal projection of a positive tilt has heading theta - pi/2. Choosing positive tilt toward +Y would be a different valid coordinate convention and would require corresponding sign changes in the entire model.

## Physical parameters

| Symbol | Meaning | Unit |
| --- | --- | --- |
| m_r | Arm mass | kg |
| m | Pendulum mass; m_p in the course brief | kg |
| r | Distance from motor axis to pendulum hinge | m |
| L_p | Full pendulum length | m |
| l | Hinge-to-center-of-mass distance; L_p/2 for a uniform rod | m |
| g | Gravitational acceleration | m/s^2 |
| J_r | Arm-side inertia about motor axis, excluding pendulum | kg m^2 |
| J_c | Transverse pendulum inertia about center of mass | kg m^2 |
| J_p | Pendulum inertia about its hinge | kg m^2 |
| b_r | Arm viscous damping coefficient | N m s/rad |
| b_p | Pendulum viscous damping coefficient | N m s/rad |
| tau | Torque applied by the motor to the arm | N m |
| R_m | Motor terminal resistance | ohm |
| k_t | Motor torque constant | N m/A |
| k_e | Motor back-EMF constant | V s/rad |
| i | Motor current | A |
| V_m | Motor voltage, the final model input | V |

## Inertia definitions

For the same uniform pendulum rod:

$$
J_c=\frac{mL_p^2}{12},\qquad J_p=J_c+ml^2=\frac{mL_p^2}{3}.
$$

Both formulas use the full rod length. The center-of-mass distance is halved, but the rod's moment of inertia is not simply divided by two. The change from J_c to J_p follows the parallel-axis theorem for two parallel transverse axes separated by l.

The course brief calls center-of-mass inertia j_p; it corresponds to J_c here. This repository reserves J_p for hinge inertia.

## Derived abbreviations

| Symbol | Definition | Purpose |
| --- | --- | --- |
| J_a | J_r + m r^2 | Combined coefficient arising when total kinetic energy is grouped |
| h | m r l | Inertial coupling coefficient |
| gamma | m g l | Gravitational torque scale |
| Delta | J_a J_p - h^2 | Determinant of the upright mechanical inertia matrix |
| b_eff | b_r + k_t k_e/R_m | Mechanical damping plus speed-proportional electrical damping |

J_a does not replace J_r in the energy of the arm alone. The arm energy is always (1/2) J_r theta_dot^2; J_a appears after adding the pendulum's translation term (1/2) m r^2 theta_dot^2.

## Assumptions

1. The arm and pendulum are rigid.
2. The pendulum is a uniform slender rod with its center at half its full length.
3. The baseline arm inertia is approximated by a uniform slender arm about an end pivot: J_r = m_r r^2/3.
4. Separate rotor and attachment-hub inertias are omitted from the numerical example.
5. The rod's inertia about its own longitudinal axis is neglected.
6. Friction is represented by viscous torques opposing velocity. Coulomb friction and stiction are not included.
7. Motor inductance is neglected, giving an algebraic current-voltage relation and four mechanical states.
8. The linear model applies to small deviations from upright at rest. It is not a large-angle swing-up model.
9. Angle and motor-voltage signs follow the stated convention; physical encoder directions have not yet been checked.

These assumptions specify the mathematical model. Model validation, simulation, and hardware experiments remain future work.
