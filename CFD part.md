# Part 2 : CFD Analysis

## 1. Introduction

Optimizing the aerodynamic performance of a PC fan requires carefully balancing key fluid dynamic parameters—primarily maximizing volumetric airflow and static pressure rise while minimizing flow separation and turbulence. Relying solely on physical prototyping to achieve these aerodynamic targets is both costly and time-consuming. Therefore, integrating Computer-Aided Design (CAD) with Computational Fluid Dynamics (CFD) is essential to the engineering workflow. 

By utilizing cloud-based platforms such as Onshape and SimScale, this iterative process becomes highly agile. Onshape enables precise 3D modeling and rapid parametric adjustments of complex blade geometries, while SimScale provides immediate cloud-based CFD simulations to analyze velocity fields, pressure distributions, and aerodynamic efficiency prior to manufacturing. This workflow significantly reduces prototyping costs, prevents material waste, and yields a faster, more reliable, and aerodynamically optimized design.

---

## 2. 2D Cascade Analysis

### 2.1 Design Targets

While the airflow inside a turbomachine is inherently three-dimensional, unsteady, and complex, it can be effectively simplified during the preliminary design stages by modeling it as a steady, two-dimensional blade cascade. Based on the preliminary mean-line sizing calculations, the objective of this section is to numerically evaluate the aerodynamic performance and efficiency of the axial fan blade section using the following target parameters:

| Parameter | Symbol | Target Value |
| :--- | :---: | :---: |
| Blade span (extrusion length) | $z$ | $35\text{ mm}$ |
| Lift coefficient | $C_L$ | $1.0$ |
| Static pressure rise | $\Delta P_s$ | $4.9\text{ Pa}$ |
| Camber angle | $\theta$ | $9.2925^\circ$ |
| Deviation angle | $\delta$ | $6.4944^\circ$ |
| Chord length | $l$ | $10.03\text{ mm}$ |
| Stagger angle | $\xi$ | $73.8111^\circ$ |
| Diffusion factor | $DF$ | $0.3765$ |
| Pitch-to-chord ratio | $s/l$ | $3.6$ |

---

### 2.2 CAD Modeling

A 4-digit NACA airfoil profile was selected for the axial fan blade due to its balance of geometric simplicity, aerodynamic robustness, and numerical stability. Because these profiles are defined by exact, smooth analytical polynomials rather than discrete tabulated points, they eliminate surface curvature discontinuities in CAD and prevent artificial pressure spikes or premature flow separation in the boundary-layer mesh. Aerodynamically, their relatively generous leading-edge radius provides high tolerance to incidence variations across off-design operating points, while their predictable behavior at low Reynolds numbers mitigates severe laminar separation bubbles. Furthermore, since classical 1D/2D turbomachinery cascade correlations were calibrated on standard airfoil families, using a 4-digit NACA profile enables direct benchmarking between preliminary analytical calculations and CFD results.

A 4-digit NACA airfoil is designated as **NACA MPXX**, where:
- **$M$**: Maximum camber in percentage of the chord length.
- **$P$**: Position of the maximum camber in tenths of the chord length.
- **$XX$**: Maximum thickness in percentage of the chord length.

From the target camber angle ($\theta = 9.2925^\circ$), the maximum camber $f_{\max}$ and first digit $M$ are calculated as:

$$
f_{\max} = \frac{l}{2} \tan\left(\frac{\theta}{4}\right)
$$

$$
M = 100 \cdot \frac{f_{\max}}{l} = 50 \tan\left(\frac{9.2925^\circ}{4}\right) \approx 2.03 \implies M = 2
$$

To remain fully consistent with the assumptions of the 2D cascade analytical model, the second digit is set to **$P = 5$**, placing the maximum camber at $50\%$ of the chord length. Both the camber angle $\theta$ and Howell's deviation correlation assume a circular-arc camber line ($a/l = 0.5$), whose geometric apex naturally lies at mid-chord.

The last two digits are set to **$XX = 12$**, corresponding to a maximum thickness-to-chord ratio of $12\%$ ($t_{\max}/l = 0.12$). Given the short chord length at the mean radius ($l = 10.03\text{ mm}$), a $12\%$ thickness yields a physical thickness of approximately $1.20\text{ mm}$, ensuring sufficient structural rigidity against aerodynamic and centrifugal loads while remaining practical to manufacture. This results in the **NACA 2512** profile.

The airfoil coordinates were generated using [Airfoil Tools (NACA 4-Digit Generator)](http://airfoiltools.com/airfoil/naca4digit) and exported as a CSV file. Before importing into Onshape, a third column containing zero values ($z = 0$) must be added to the CSV file so that Onshape's *3D Fit Spline* / routing curve tool can properly parse the 3D point cloud. Once imported (in millimeters), the profile was scaled uniformly by a factor of **$10.03$** to match the target chord length ($l = 10.03\text{ mm}$), filled into a planar surface, and extruded (thickened) to a span of **$z = 0.035\text{ m}$ ($35\text{ mm}$)**, as shown below.

| Scaling the NACA 2512 Profile ($l = 10.03\text{ mm}$) | Extruding the Blade Span ($z = 35\text{ mm}$) |
| :---: | :---: |
| ![Blade Chord Scaling](<Release_01_Images/Photo of the scale of the blade.png>) | ![Blade Span Extrusion](<Release_01_Images/photo of the thickness of the blade.png>) |

Finally, the CAD geometry was imported into SimScale and rotated around the spanwise $z$-axis to apply the required stagger angle ($\xi = 73.8111^\circ$).

---

### 2.3 CFD Setup and Simulation

#### 2.3.1 Computational Domain and Meshing

Generating a high-quality computational mesh is foundational to achieving accurate, stable, and reliable CFD results, as spatial discretization governs how accurately the Navier-Stokes equations are resolved. Proper refinement in high-gradient regions—such as boundary layers, leading and trailing edges, and downstream wakes—is essential to capture wall shear stresses, pressure distributions, and potential flow separation without excessive numerical diffusion.

For this 2D cascade simulation, SimScale's **Hex-dominant Parametric** meshing algorithm was selected because it provides granular control over local discretization while minimizing numerical diffusion through flow-aligned hexahedral cells:

1. **Background Mesh Box & Domain Dimensions:** 
   A background mesh box was defined with bounding coordinates from $\min(-0.3, -0.018, 0)\text{ m}$ to $\max(0.9, 0.018, 0.035)\text{ m}$ and a base cell resolution of $(100, 3, 3)$ to maintain near-cubic cells.
   - **Pitchwise ($y$-direction):** Bounded between $-s/2$ and $+s/2$ ($\pm 0.018\text{ m}$, matching $s = 3.6 \times l \approx 0.036\text{ m}$) around the centered airfoil. Applying translational periodic boundary conditions to the upper and lower faces replicates the aerodynamic interference of an infinite cascade without meshing the full rotor.
   - **Streamwise ($x$-direction):** Extended far upstream and downstream of the blade so that the inflow remains uniform and undisturbed by the airfoil's upstream potential field, while allowing full wake dissipation and pressure recovery before the outlet.
   - **Spanwise ($z$-direction):** Constrained directly to the two spanwise faces of the extruded section ($0$ to $0.035\text{ m}$) with symmetry boundary conditions to enforce a strictly two-dimensional flow regime free of 3D tip-vortex or end-wall effects.

2. **Local Refinements:** 
   A **Level 5** surface and region refinement was applied around the airfoil and its immediate wake to resolve steep velocity and pressure gradients near the leading and trailing edges while keeping the far-field mesh coarse.

![Computational Domain and Refinement Box](<Release_01_Images/Mesh box blue.png>)

3. **Boundary Layer Inflation & $y^+$ Calculation:**
   To capture boundary-layer dynamics while optimizing computational cost via standard wall functions, the first cell center must lie within the logarithmic region ($y^+ \ge 30$).

![Dimensionless Wall Distance y+ Regions](<Release_01_Images/y+ schemes.png>)

The dimensionless wall distance $y^+$ is defined as:

$$
y^+ = \frac{y \, u_\tau}{\nu} \tag{1}
$$

where $y$ is the normal distance from the wall to the first cell center, $\nu = 1.529 \times 10^{-5}\text{ m}^2/\text{s}$ is the kinematic viscosity of air, and $u_\tau$ is the friction velocity:

$$
u_\tau = \sqrt{\frac{\tau_w}{\rho}} \tag{2}
$$

With air density $\rho = 1.196\text{ kg/m}^3$, relative inlet velocity $W_1 = 9.11\text{ m/s}$, and chord length $c \approx 0.01\text{ m}$, the chord-based Reynolds number is:

$$
\text{Re}_c = \frac{W_1 \, c}{\nu} = \frac{9.11 \times 0.01}{1.529 \times 10^{-5}} \approx 5\,958
$$

Using the flat-plate turbulent boundary-layer correlation for the skin friction coefficient $C_f$:

$$
C_f = \frac{0.058}{\text{Re}_c^{0.2}} = \frac{0.058}{(5958)^{0.2}} \approx 0.0102
$$

The wall shear stress $\tau_w$ and friction velocity $u_\tau$ are:

$$
\tau_w = \frac{1}{2} \rho W_1^2 C_f = 0.5 \times 1.196 \times (9.11)^2 \times 0.0102 \approx 0.506\text{ Pa}
$$

$$
u_\tau = \sqrt{\frac{0.506}{1.196}} \approx 0.650\text{ m/s}
$$

Rearranging Equation (1) for $y^+ \gt 30$:

$$
y = \frac{y^+ \, \nu}{u_\tau} \gt \frac{30 \times 1.529 \times 10^{-5}}{0.650} \approx 7.05 \times 10^{-4}\text{ m}
$$

Since $y$ represents the distance to the **center** of the first cell, the total height of the first boundary-layer cell ($\Delta y = 2y$) must satisfy $\Delta y \gt 1.41 \times 10^{-3}\text{ m}$ ($1.41\text{ mm}$). The boundary-layer inflation in SimScale was therefore configured with:
- **Number of layers:** $2$
- **Expansion ratio:** $1.2$
- **Minimum first layer thickness ($\Delta y$):** $1.5 \times 10^{-3}\text{ m}$ ($1.5\text{ mm}$)
- **Final layer thickness:** $0.5$ (relative)

This yields an actual dimensionless wall distance of:

$$
y^+ = \frac{(\Delta y / 2) \, u_\tau}{\nu} = \frac{0.75 \times 10^{-3} \times 0.650}{1.529 \times 10^{-5}} \approx 31.9 \gt 30
$$

confirming that the first cell center lies just inside the log-law region.

| Global Domain Mesh | Local Airfoil and Boundary Layer Mesh |
| :---: | :---: |
| ![Global Mesh](<Release_01_Images/mesh global.png>) | ![Local Airfoil Mesh](<Release_01_Images/mesh airfoil.png>) |

---

#### 2.3.2 Boundary Conditions

To simulate steady, incompressible airflow through the 2D blade cascade, the following boundary conditions were applied:
- **Velocity Inlet ($x_{\min}$):** Fixed uniform relative velocity vector $\vec{W}_1 = (2.82,\; 8.66,\; 0)\text{ m/s}$, corresponding to a magnitude of $W_1 = \sqrt{2.82^2 + 8.66^2} = 9.11\text{ m/s}$ and an inlet flow angle $\beta_1 = \arctan(8.66 / 2.82) = 71.96^\circ$.
- **Pressure Outlet ($x_{\max}$):** Fixed static gauge pressure of $P_{\text{out}} = 4.9\text{ Pa}$.
- **Periodic Faces ($y_{\min}, y_{\max}$):** Translational periodic boundary conditions matching the blade pitch ($s = 0.036\text{ m}$).
- **Lateral Faces ($z_{\min}, z_{\max}$):** Symmetry boundary conditions to enforce 2D flow behavior.
- **Airfoil Surface:** No-slip wall boundary condition with standard wall functions.

---

#### 2.3.3 Flow Field Analysis

![Velocity Magnitude Contour](<Release_01_Images/velocity magnitude.png>)

The velocity magnitude contour illustrates the flow deflection and acceleration around the NACA 2512 airfoil. A distinct high-velocity region (in red/orange, peaking at $11.08\text{ m/s}$) develops over the suction side just downstream of the leading edge, indicating the local static pressure drop responsible for lift generation. Conversely, the flow decelerates sharply at the leading-edge stagnation point and within the boundary layer along the airfoil wall (in blue). Downstream of the trailing edge, a well-defined wake (in green/yellow) exhibits a clear momentum deficit, while the parallel oblique bands confirm the periodicity across adjacent blade passages.

**Physical Consistency:**
- **Flow Turning:** The downstream wake visibly deflects upward compared to the incoming flow angle, confirming that the blade successfully imparts angular momentum to the fluid.
- **Pressure–Suction Asymmetry:** Strong acceleration along the upper (suction) surface contrasted with lower velocities along the lower (pressure) surface confirms positive aerodynamic loading.
- **Boundary Layer & Periodicity:** Smooth boundary-layer attachment without massive stall and seamless continuity of the periodic wakes validate both the angle of attack and the numerical setup.

By extracting the area-averaged streamwise ($W_{2,x}$) and pitchwise ($W_{2,y}$) velocity components downstream of the trailing edge, the outlet relative flow angle $\beta_2$ is computed as:

$$
\beta_2 = \arctan\left(\frac{W_{2,y}}{W_{2,x}}\right)
$$

---

#### 2.3.4 Results and Discussion

| Parameter | Analytical Theory ($C_L = 1.0$) | CFD Simulation | Deviation |
| :--- | :---: | :---: | :---: |
| **Inlet relative angle ($\beta_1$)** | $71.96^\circ$ | $71.96^\circ$ | $0.00^\circ$ |
| **Outlet relative angle ($\beta_2$)** | $69.16^\circ$ | $70.03^\circ$ | $+0.87^\circ$ |
| **Flow deflection ($\epsilon = \beta_1 - \beta_2$)** | $2.80^\circ$ | $1.93^\circ$ | $-0.87^\circ$ |
| **Inlet relative velocity ($W_1$)** | $9.11\text{ m/s}$ | $9.11\text{ m/s}$ | $0.00\text{ m/s}$ |
| **Outlet relative velocity ($W_2$)** | $7.93\text{ m/s}$ | $8.26\text{ m/s}$ | $+0.33\text{ m/s}$ |
| **Lift coefficient ($C_L$)** | $1.000$ | $0.536$ | $-46.4\%$ |
| **Drag coefficient ($C_D$)** | $0.0200$ | $0.0746$ | $+273\%$ |
| **Diffusion factor ($DF$)** | $0.3765$ | $0.2510$ | $-33.3\%$ |
| **Static pressure rise ($\Delta p$)** | $10.83\text{ Pa}$ | $6.03\text{ Pa}$ | $-44.3\%$ |
| **Hydraulic efficiency ($\eta_h$)** | $100\%$ *(ideal assumption)* | $59.3\%$ | $-40.7\%$ |

The discrepancies between the analytical predictions and the CFD results are primarily driven by **low Reynolds number viscous effects** ($\text{Re}_c \approx 6\,000$) and the limitations of classical empirical cascade correlations:

1. **Boundary-Layer Thickening and Increased Drag:** Howell's deviation rule and the preliminary drag estimate ($C_D \approx 0.02$) were derived from wind-tunnel cascade tests operating at much higher Reynolds numbers ($\text{Re}_c \gt 10^5$), where boundary layers remain thin and energized. At $\text{Re}_c \approx 6\,000$, viscous forces dominate, significantly thickening the suction-side boundary layer and sharply increasing skin friction and form drag ($C_D = 0.0746$ vs. $0.0200$).
2. **Reduced Effective Camber and Flow Turning:** This viscous thickening alters the effective aerodynamic camber of the airfoil and increases the outlet deviation angle ($\delta$), reducing the actual flow deflection ($\beta_2 = 70.03^\circ$ instead of $69.16^\circ$, giving $\epsilon = 1.93^\circ$).
3. **Drop in Lift and Pressure Rise:** Because Euler's turbomachinery equation directly links pressure rise to the change in tangential velocity ($\Delta c_\theta$), the reduced flow turning leads to a proportional drop in lift coefficient ($C_L = 0.536$), diffusion factor ($DF = 0.251$), and actual static pressure rise ($\Delta p = 6.03\text{ Pa}$ vs. $10.83\text{ Pa}$).
4. **Numerical Modeling Consideration:** Additionally, at $\text{Re}_c \approx 6\,000$, the boundary layer is largely transitional or laminar rather than fully turbulent. Using a high-$y^+$ wall-function mesh ($y^+ \approx 31.9$, where the first cell height of $1.5\text{ mm}$ represents $15\%$ of the $10\text{ mm}$ chord) introduces extra numerical dissipation near the wall. In future iterations, resolving the viscous sublayer directly ($y^+ \lt 1$) with a low-Reynolds turbulence model (such as $k\text{-}\omega\text{ SST}$) would capture the near-wall velocity profile with even greater fidelity.
