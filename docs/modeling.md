# Derivation of the rotary pendulum model

This document follows the physical and mathematical reasoning used to obtain the upright model. The order matters: define the geometry, calculate velocities, calculate energies, apply Euler-Lagrange, linearize the resulting equations, and finally collect state-space coefficients. The work here concerns modeling; no controller has been designed and no simulation or hardware validation is claimed.

## 1. Coordinates, geometry, and conventions

Use the motor axis at hinge height as the origin. X and Y are horizontal; Z points upward. Let P be the pendulum hinge and G the pendulum center of mass.

- theta: arm angle, positive counterclockwise viewed from above.
- alpha: pendulum tilt, zero upright. Positive tilt has a horizontal projection in the negative tangential direction of arm rotation.
- r: motor-axis to hinge distance.
- l: hinge to pendulum center-of-mass distance; l = L_p/2 for a uniform rod.

At theta = 0 the arm points along +X and positive alpha tilts toward -Y. The pendulum swings in the vertical plane perpendicular to the arm. The horizontal projection of the rod therefore has signed heading theta - pi/2. Measuring the same clockwise angle as pi/2 - theta gives identical components when the Y sign is included.

The hinge travels in a circle parallel to the top of the QUBE:

$$
X_P=r\cos\theta,\qquad Y_P=r\sin\theta,\qquad Z_P=0.
$$

The center-of-mass position follows by adding the rod projections to P:

$$
\begin{aligned}
X_G&=r\cos\theta+l\sin\alpha\sin\theta,\\
Y_G&=r\sin\theta-l\sin\alpha\cos\theta,\\
Z_G&=l\cos\alpha.
\end{aligned}
$$

These signs define this repository's model. Reversing an encoder's alpha direction requires a consistent coordinate transformation of the equations and controller.

## 2. Center-of-mass velocity

Kinetic energy requires speed. Differentiate position with respect to time, using the product and chain rules:

The lengths r and l are constants, while theta and alpha depend on time. In particular:

$$
\frac{d}{dt}\cos\theta=-\sin\theta\dot\theta,
$$

$$
\frac{d}{dt}(\sin\alpha\sin\theta)=\cos\alpha\dot\alpha\sin\theta+\sin\alpha\cos\theta\dot\theta.
$$

$$
\begin{aligned}
\dot X_G&=-r\sin\theta\dot\theta+l\cos\alpha\sin\theta\dot\alpha+l\sin\alpha\cos\theta\dot\theta,\\
\dot Y_G&=r\cos\theta\dot\theta-l\cos\alpha\cos\theta\dot\alpha+l\sin\alpha\sin\theta\dot\theta,\\
\dot Z_G&=-l\sin\alpha\dot\alpha.
\end{aligned}
$$

Because the Cartesian components are perpendicular:

$$
v_G^2=\dot X_G^2+\dot Y_G^2+\dot Z_G^2.
$$

First combine the horizontal components:

$$
\dot X_G^2+\dot Y_G^2=(r\dot\theta-l\cos\alpha\dot\alpha)^2+l^2\sin^2\alpha\dot\theta^2.
$$

To see the cancellation explicitly, set a = r theta_dot - l cos(alpha) alpha_dot and b = l sin(alpha) theta_dot. The derivatives are X_dot = -a sin(theta) + b cos(theta) and Y_dot = a cos(theta) + b sin(theta). Their squared sum is a^2 + b^2 because the mixed terms cancel and sin^2(theta) + cos^2(theta) = 1. These temporary a and b are just algebraic abbreviations, not new physical parameters.

Adding the vertical component and using sin-squared plus cos-squared equals one gives:

$$
v_G^2=(r^2+l^2\sin^2\alpha)\dot\theta^2+l^2\dot\alpha^2-2rl\cos\alpha\dot\theta\dot\alpha.
$$

The cross term follows from squaring the sum of velocities. It expresses how the hinge movement and the rod's tilt movement can reinforce or oppose one another. Its negative sign follows from the chosen positive alpha direction.

## 3. Inertias and assumptions

For a uniform slender pendulum:

$$
J_c=\frac{mL_p^2}{12},\qquad J_p=J_c+ml^2=\frac{mL_p^2}{3}.
$$

These inertia formulas follow from the definition J = integral of squared perpendicular distance times mass. For a uniform rod measured from its center, dm = (m/L_p) ds, and:

$$
J_c=\frac{m}{L_p}\int_{-L_p/2}^{L_p/2}s^2\,ds=\frac{mL_p^2}{12}.
$$

For a uniform arm measured from its end pivot:

$$
J_r\simeq\frac{m_r}{r}\int_0^r s^2\,ds=\frac{m_r r^2}{3}.
$$

The two axes used in the pendulum parallel-axis theorem are perpendicular to the rod, mutually parallel, and separated by l. For l = L_p/2:

$$
J_p=\frac{mL_p^2}{12}+m\left(\frac{L_p}{2}\right)^2=\frac{mL_p^2}{3}.
$$

J_c is the transverse inertia about G. J_p is the inertia about the hinge, found using the parallel-axis theorem. The course brief's center-of-mass `j_p` is this repository's J_c.

J_r is the arm-side inertia about the motor axis, excluding the pendulum. For the baseline numerical model it is approximated as m_r r^2/3; separate rotor and hub inertias are omitted. A more detailed model can include those contributions in J_r without changing the derivation.

The pendulum is treated as a slender rod with negligible axial inertia. Friction is modeled as viscous damping. The four-state voltage model neglects motor inductance, so motor current is not an additional state.

## 4. Kinetic energy

For a rigid body, separate translation of G from rotation about G:

$$
T_{\rm arm}=\frac12J_r\dot\theta^2,
$$

$$
T_{\rm pend,trans}=\frac12m\left[(r^2+l^2\sin^2\alpha)\dot\theta^2+l^2\dot\alpha^2-2rl\cos\alpha\dot\theta\dot\alpha\right],
$$

$$
T_{\rm pend,rot}=\frac12J_c\left(\dot\alpha^2+\dot\theta^2\sin^2\alpha\right).
$$

For the last expression, consider mass dm at signed distance s from G along the rod. After removing G's translation, its distance from the vertical axis through G is |s| sin(alpha). Its speed due to arm rotation is |s sin(alpha) theta_dot|. Integrating its energy gives:

$$
T_{\theta,G}=\frac12\dot\theta^2\sin^2\alpha\int s^2\,dm=\frac12J_c\sin^2\alpha\dot\theta^2.
$$

Equivalently, define J_Gz as the rod's instantaneous inertia about the vertical through G. Then J_Gz = J_c sin^2(alpha), and T_theta,G = (1/2) J_Gz theta_dot^2. The angular rate around that vertical is theta_dot; the sine-squared factor describes how the mass distribution relative to the vertical changes with tilt.

Tilting contributes (1/2) J_c alpha_dot^2. These two relative velocity contributions are perpendicular, so they add in energy. Use J_c here because translation of G has already been included.

Expand every term before grouping:

$$
\begin{aligned}
T={}&\frac12J_r\dot\theta^2+\frac12mr^2\dot\theta^2+\frac12ml^2\sin^2\alpha\dot\theta^2\\
&+\frac12ml^2\dot\alpha^2-mrl\cos\alpha\dot\theta\dot\alpha\\
&+\frac12J_c\dot\alpha^2+\frac12J_c\sin^2\alpha\dot\theta^2.
\end{aligned}
$$

Define abbreviations:

$$
J_a=J_r+mr^2,\qquad J_p=J_c+ml^2,\qquad h=mrl,\qquad \gamma=mgl.
$$

Then the exact grouped energy within this model is:

$$
T=\frac12(J_a+J_p\sin^2\alpha)\dot\theta^2+\frac12J_p\dot\alpha^2-h\cos\alpha\dot\theta\dot\alpha.
$$

No translation terms have been removed. J_a is an abbreviation in the total energy; the arm alone still has energy (1/2) J_r theta_dot^2.

## 5. Potential energy and Lagrangian

With zero gravitational potential at hinge height:

$$
V=mgZ_G=mgl\cos\alpha=\gamma\cos\alpha.
$$

The arm has constant height, so its potential contributes only a constant, which disappears from derivatives. Upright has maximum pendulum gravitational potential.

$$
\mathcal L=T-V.
$$

## 6. Euler-Lagrange equations

Apply one equation for each independent coordinate q = theta, alpha:

$$
\frac{d}{dt}\left(\frac{\partial\mathcal L}{\partial\dot q}\right)-\frac{\partial\mathcal L}{\partial q}=Q_q.
$$

The partial derivatives temporarily treat positions and velocities as independent arguments of the Lagrangian. The subsequent total time derivative restores their time dependence. This is why the differentiation generates angular accelerations and products of angular velocities. Since the generalized coordinates are angles, the corresponding Q quantities have units of torque.

The non-conservative generalized torques are:

$$
Q_\theta=\tau-b_r\dot\theta,\qquad Q_\alpha=-b_p\dot\alpha.
$$

The motor acts directly on theta. Each friction torque opposes the corresponding velocity. Gravity is already represented by V and is not added again to Q.

For theta:

$$
\frac{\partial\mathcal L}{\partial\dot\theta}=(J_a+J_p\sin^2\alpha)\dot\theta-h\cos\alpha\dot\alpha,\qquad \frac{\partial\mathcal L}{\partial\theta}=0.
$$

Differentiate the first expression in time:

$$
\begin{aligned}
\frac{d}{dt}\frac{\partial\mathcal L}{\partial\dot\theta}
={}&(J_a+J_p\sin^2\alpha)\ddot\theta-h\cos\alpha\ddot\alpha\\
&+2J_p\sin\alpha\cos\alpha\dot\theta\dot\alpha+h\sin\alpha\dot\alpha^2.
\end{aligned}
$$

For alpha:

$$
\frac{\partial\mathcal L}{\partial\dot\alpha}=J_p\dot\alpha-h\cos\alpha\dot\theta,
$$

$$
\frac{d}{dt}\frac{\partial\mathcal L}{\partial\dot\alpha}=J_p\ddot\alpha-h\cos\alpha\ddot\theta+h\sin\alpha\dot\alpha\dot\theta,
$$

$$
\frac{\partial\mathcal L}{\partial\alpha}=J_p\sin\alpha\cos\alpha\dot\theta^2+h\sin\alpha\dot\theta\dot\alpha+\gamma\sin\alpha.
$$

Subtracting cancels the matching mixed-velocity terms. The nonlinear equations are:

$$
\begin{aligned}
&(J_a+J_p\sin^2\alpha)\ddot\theta-h\cos\alpha\ddot\alpha\\
&\quad+2J_p\sin\alpha\cos\alpha\dot\theta\dot\alpha+h\sin\alpha\dot\alpha^2=\tau-b_r\dot\theta,
\end{aligned}
$$

$$
J_p\ddot\alpha-h\cos\alpha\ddot\theta-J_p\sin\alpha\cos\alpha\dot\theta^2-\gamma\sin\alpha=-b_p\dot\alpha.
$$

## 7. Upright linearization

Linearize at theta = 0 (chosen arm reference), alpha = 0, both velocities zero, and zero torque. Keep first-order disturbances:

$$
\sin\alpha\simeq\alpha,\qquad \cos\alpha\simeq1.
$$

Terms such as alpha-squared times theta acceleration and alpha times velocity products are higher order. Drop them only at this linearization stage. Keep gravity proportional to alpha and viscous damping proportional to velocity.

The exact term-by-term substitutions are:

| Nonlinear term | First-order result | Reason |
| --- | --- | --- |
| (J_a + J_p sin^2(alpha)) theta_ddot | J_a theta_ddot | The extra term contains alpha^2 times a disturbance acceleration |
| -h cos(alpha) alpha_ddot | -h alpha_ddot | cos(alpha) is one to first order |
| 2 J_p sin(alpha) cos(alpha) theta_dot alpha_dot | 0 | Product of small deviations |
| h sin(alpha) alpha_dot^2 | 0 | Product of small deviations |
| -h cos(alpha) theta_ddot | -h theta_ddot | cos(alpha) is one to first order |
| -J_p sin(alpha) cos(alpha) theta_dot^2 | 0 | Product of small deviations |
| -gamma sin(alpha) | -gamma alpha | The first-order gravity term must remain |

For example, if every disturbance is scaled by a small epsilon, alpha theta_dot alpha_dot scales as epsilon^3, while alpha and each velocity scale as epsilon. Linearization retains the terms proportional to epsilon. The neglected terms are not physically zero; they are absent only from this local first-order approximation.

$$
J_a\ddot\theta-h\ddot\alpha+b_r\dot\theta=\tau,
$$

$$
-h\ddot\theta+J_p\ddot\alpha+b_p\dot\alpha-\gamma\alpha=0.
$$

Holding the arm fixed and ignoring pendulum damping gives J_p alpha_ddot = gamma alpha: a small tilt grows, consistent with upright instability.

## 8. State equations by scalar substitution

Define:

$$
x=[\theta,\alpha,\dot\theta,\dot\alpha]^T.
$$

Therefore x1_dot = x3 and x2_dot = x4. The other equations are:

$$
J_a\dot x_3-h\dot x_4+b_rx_3=\tau,
$$

$$
-h\dot x_3+J_p\dot x_4+b_px_4-\gamma x_2=0.
$$

Isolate x3_dot in the first equation:

$$
\dot x_3=(\tau+h\dot x_4-b_rx_3)/J_a.
$$

Substitute into the second and multiply by J_a:

$$
-h\tau-h^2\dot x_4+hb_rx_3+J_aJ_p\dot x_4+J_ab_px_4-J_a\gamma x_2=0.
$$

Let Delta = J_a J_p - h^2. Solve for x4_dot and substitute back for x3_dot:

$$
\dot x_3=\frac{h\gamma}{\Delta}x_2-\frac{J_pb_r}{\Delta}x_3-\frac{hb_p}{\Delta}x_4+\frac{J_p}{\Delta}\tau,
$$

$$
\dot x_4=\frac{J_a\gamma}{\Delta}x_2-\frac{hb_r}{\Delta}x_3-\frac{J_ab_p}{\Delta}x_4+\frac{h}{\Delta}\tau.
$$

For completeness, substituting x4_dot back into x3_dot gives:

$$
\dot x_3=\frac{\Delta(\tau-b_rx_3)+h(h\tau-hb_rx_3-J_ab_px_4+J_a\gamma x_2)}{J_a\Delta}.
$$

The torque coefficient simplifies because Delta + h^2 = J_a J_p. The same identity simplifies the x3 coefficient. The remaining numerator terms each contain J_a, which cancels the J_a in the denominator, producing the x3_dot expression above. No derivative remains on the right-hand side.

This is ordinary simultaneous-equation substitution; a matrix inverse is an equivalent calculation.

## 9. Motor voltage input

Neglect motor inductance. Kirchhoff's voltage law and the torque-current relation give:

$$
V_m=R_mi+k_e\dot\theta,\qquad \tau=k_ti.
$$

Eliminate i:

$$
\tau=\frac{k_t}{R_m}V_m-\frac{k_tk_e}{R_m}\dot\theta.
$$

Here k_t converts current to torque, and k_e converts motor speed to back-EMF voltage. Substituting this into the linear arm equation gives:

$$
J_a\ddot\theta-h\ddot\alpha+b_r\dot\theta=\frac{k_t}{R_m}V_m-\frac{k_tk_e}{R_m}\dot\theta.
$$

Move the back-EMF term to the left and collect the coefficients of theta_dot:

$$
J_a\ddot\theta-h\ddot\alpha+\left(b_r+\frac{k_tk_e}{R_m}\right)\dot\theta=\frac{k_t}{R_m}V_m.
$$

Move the speed-proportional term to the left of the arm equation. Its coefficient combines with mechanical damping:

$$
b_{\rm eff}=b_r+\frac{k_tk_e}{R_m}.
$$

For u = V_m:

$$
\dot x_3=\frac{h\gamma}{\Delta}x_2-\frac{J_pb_{\rm eff}}{\Delta}x_3-\frac{hb_p}{\Delta}x_4+\frac{J_pk_t}{R_m\Delta}u,
$$

$$
\dot x_4=\frac{J_a\gamma}{\Delta}x_2-\frac{hb_{\rm eff}}{\Delta}x_3-\frac{J_ab_p}{\Delta}x_4+\frac{hk_t}{R_m\Delta}u.
$$

## 10. State-space matrices

Read off the coefficient of each state in each equation:

$$
A=\begin{bmatrix}
0&0&1&0\\0&0&0&1\\
0&h\gamma/\Delta&-J_pb_{\rm eff}/\Delta&-hb_p/\Delta\\
0&J_a\gamma/\Delta&-hb_{\rm eff}/\Delta&-J_ab_p/\Delta
\end{bmatrix},\qquad
B=\begin{bmatrix}0\\0\\J_pk_t/(R_m\Delta)\\hk_t/(R_m\Delta)\end{bmatrix}.
$$

For measured angle outputs y = [theta, alpha]^T:

$$
C=\begin{bmatrix}1&0&0&0\\0&1&0&0\end{bmatrix},\qquad D=\begin{bmatrix}0\\0\end{bmatrix}.
$$

C selects the measured angle states. D is zero because voltage does not instantaneously change position. The zero first column of A reflects rotational symmetry: arm heading itself does not enter these motion equations.

The first two rows of A express x1_dot = x3 and x2_dot = x4. The third and fourth rows are the coefficients of x1, x2, x3, x4 in the two acceleration equations. B contains the coefficient of voltage in each equation; its first two entries are zero because voltage first affects acceleration rather than producing an instantaneous angle derivative term.

All equations in this document are a mathematical model. Their derivation does not establish that a physical pendulum has been stabilized. Experimental validation and control design are separate future steps.
