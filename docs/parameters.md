# Numerical substitution and matrix calculations

This section evaluates the equations already derived. It records calculated values only; it is not a simulation or experimental result.

## Parameter sources

The course brief specifies QUBE-Servo 3, upright alpha = 0, voltage input, angle outputs, and b_r = 1e-3, b_p = 5e-5 N m s/rad. The remaining hardware constants below are from the manufacturer-authored [QUBE-Servo 3 User Manual, Table 2.2](https://www.made-for-science.com/en/quanser/?df=made-for-science-quanser-qube-servo-3-usermanual.pdf).

| Parameter | Value | Unit |
| --- | ---: | --- |
| Arm mass m_r | 0.095 | kg |
| Arm length r | 0.085 | m |
| Pendulum mass m | 0.024 | kg |
| Full pendulum length L_p | 0.129 | m |
| Center-of-mass distance l = L_p/2 | 0.0645 | m |
| Motor resistance R_m | 7.5 | ohm |
| Torque constant k_t | 0.0422 | N m/A |
| Back-EMF constant k_e | 0.0422 | V s/rad |
| Arm damping b_r | 0.001 | N m s/rad |
| Pendulum damping b_p | 0.00005 | N m s/rad |
| Assumed gravity g | 9.81 | m/s^2 |

The baseline assumes uniform slender links and omits separate motor/hub inertia and motor inductance. The brief's center-of-mass j_p is J_c here; J_p is hinge inertia.

## Calculate the inertia and coupling constants

$$
J_r=\frac{m_r r^2}{3}=\frac{0.095(0.085)^2}{3}=2.2879166667\times10^{-4}\ \mathrm{kg\,m^2}.
$$

$$
J_c=\frac{mL_p^2}{12}=\frac{0.024(0.129)^2}{12}=3.3282\times10^{-5}\ \mathrm{kg\,m^2}.
$$

$$
J_p=J_c+ml^2=3.3282\times10^{-5}+0.024(0.0645)^2=1.33128\times10^{-4}\ \mathrm{kg\,m^2}.
$$

$$
J_a=J_r+mr^2=2.2879166667\times10^{-4}+0.024(0.085)^2=4.0219166667\times10^{-4}\ \mathrm{kg\,m^2}.
$$

$$
h=mrl=0.024(0.085)(0.0645)=1.3158\times10^{-4}\ \mathrm{kg\,m^2}.
$$

$$
\gamma=mgl=0.024(9.81)(0.0645)=0.01518588\ \mathrm{N\,m}.
$$

$$
\Delta=J_aJ_p-h^2=3.62296758\times10^{-8}\ (\mathrm{kg\,m^2})^2.
$$

## Include the voltage input

The torque-current law and the motor voltage equation, neglecting inductance, are:

$$
\tau=k_ti,\qquad V_m=R_mi+k_e\dot\theta.
$$

Solve for current and substitute:

$$
\tau=\frac{k_t}{R_m}V_m-\frac{k_tk_e}{R_m}\dot\theta
=0.0056266667V_m-0.0002374453\dot\theta.
$$

The speed-proportional back-EMF term adds to the arm damping:

$$
b_{\mathrm{eff}}=b_r+\frac{k_tk_e}{R_m}
=0.001+\frac{0.0422(0.0422)}{7.5}
=0.0012374453333\ \mathrm{N\,m\,s/rad}.
$$

## Calculate every nontrivial entry in A

The state order is theta, alpha, theta_dot, alpha_dot. The first two rows follow from x1_dot = x3 and x2_dot = x4. The first column is zero because neither acceleration equation contains theta itself.

$$
A_{32}=\frac{h\gamma}{\Delta}
=\frac{(1.3158\times10^{-4})(0.01518588)}{3.62296758\times10^{-8}}
=55.152524727.
$$

$$
A_{33}=-\frac{J_pb_{\mathrm{eff}}}{\Delta}
=-\frac{(1.33128\times10^{-4})(0.0012374453333)}{3.62296758\times10^{-8}}
=-4.547063111.
$$

$$
A_{34}=-\frac{hb_p}{\Delta}
=-\frac{(1.3158\times10^{-4})(5\times10^{-5})}{3.62296758\times10^{-8}}
=-0.181591468.
$$

$$
A_{42}=\frac{J_a\gamma}{\Delta}
=\frac{(4.0219166667\times10^{-4})(0.01518588)}{3.62296758\times10^{-8}}
=168.580983742.
$$

$$
A_{43}=-\frac{hb_{\mathrm{eff}}}{\Delta}
=-\frac{(1.3158\times10^{-4})(0.0012374453333)}{3.62296758\times10^{-8}}
=-4.494190284.
$$

$$
A_{44}=-\frac{J_ab_p}{\Delta}
=-\frac{(4.0219166667\times10^{-4})(5\times10^{-5})}{3.62296758\times10^{-8}}
=-0.555058330.
$$

## Calculate B, C, and D

The coefficients multiplying voltage are:

$$
B_3=\frac{J_pk_t}{R_m\Delta}
=\frac{(1.33128\times10^{-4})(0.0422)}{7.5(3.62296758\times10^{-8})}
=20.675506017,
$$

$$
B_4=\frac{hk_t}{R_m\Delta}
=\frac{(1.3158\times10^{-4})(0.0422)}{7.5(3.62296758\times10^{-8})}
=20.435093156.
$$

The first two B entries are zero. The angle outputs select x1 and x2, so C selects those states. D is zero because there is no instantaneous voltage-to-angle feedthrough.

## Final numerical matrices

$$
A=\begin{bmatrix}
0&0&1&0\\
0&0&0&1\\
0&55.152525&-4.547063&-0.181591\\
0&168.580984&-4.494190&-0.555058
\end{bmatrix},\qquad
B=\begin{bmatrix}0\\0\\20.675506\\20.435093\end{bmatrix}.
$$

$$
C=\begin{bmatrix}1&0&0&0\\0&1&0&0\end{bmatrix},\qquad
D=\begin{bmatrix}0\\0\end{bmatrix}.
$$

Use radians, seconds, and volts. These matrices are calculated for the stated angle signs and the upright equilibrium. Displayed coefficients are rounded after calculating with the unrounded constants.

Using b_r alone instead of b_eff produces the torque-input model's mechanical damping coefficients. For a voltage-input model, the back-EMF contribution belongs in A and the factor k_t/R_m belongs in B.
