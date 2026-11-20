# BIT 2118: Scientific Computing — Individual Repository

**Author:** Linton Nyakundi Onsongo  
**Course:** BIT 2118: Scientific Computing  

This repository contains the Python implementations (`code/`), numerical analysis, parameter sensitivity experiments, output interpretations, and modified real-world applications for Tasks 02 through 20[cite: 1, 2, 3, 4].

---

## Task 02: NumPy Arrays and Numerical Computation (`code/Task_02_numpy_arrays.py`)[cite: 4, 5]
* **B. Inputs & Mathematical Operation:** Takes a 1D NumPy array of 24 hourly Celsius temperatures (`temperatures_c`)[cite: 5]. Computes descriptive statistics (`np.mean`, `np.min`, `np.max`) and applies the vectorized linear transformation $F = 1.8C + 32$ across all elements simultaneously[cite: 5].
* **C. Parameter Change & Effect:** Increasing the midday peak reading from `29.4` to `34.4` °C raises the daily mean from `22.77` °C to `22.98` °C and shifts the maximum temperature to `34.40` °C (`93.92` °F), while leaving `min_temp` unchanged at `17.50` °C[cite: 5].
* **D. Output Interpretation:** The daily mean (`22.77` °C) captures the average thermal baseline over 24 hours, bounded by an overnight low of `17.50` °C and an afternoon peak of `29.40` °C, converted directly into Fahrenheit (`65.30` °F for the first reading)[cite: 5].
* **E. Modified Real-World Case (Data Center Rack Thermal Monitoring):**
  * **Assumptions:** Temperatures represent hourly server-rack exhaust readings; thermal throttling triggers above `85.0` °F (`29.44` °C).
  * **Modified Input & Output:** Using rack readings `[24.0, 26.5, 28.0, 31.2, 33.0, 29.8]` °C, vectorized conversion yields `[75.2, 79.7, 82.4, 88.16, 91.4, 85.64]` °F, flagging 3 hours above the `85.0` °F cooling threshold.

---

## Task 03: Mathematical Functions and Vectorization (`code/Task_03_vectorization.py`)[cite: 5, 6]
* **B. Inputs & Mathematical Operation:** Generates $N = 1,000,000$ uniformly distributed random latency values in $[5, 250]$ ms using `rng.uniform(5, 250, 1_000_000)`[cite: 6]. Divides each element by `1000.0` to convert to seconds and squares the result ($s^2$), comparing a Python `for`-loop against vectorized NumPy array arithmetic[cite: 6].
* **C. Parameter Change & Effect:** Increasing sample size `N` from `1_000_000` to `5_000_000` scales Python loop execution time linearly (~5x slower), whereas NumPy's contiguous C-level memory operations maintain roughly a 30x–60x speed-up[cite: 6].
* **D. Output Interpretation:** Both functions return identical squared-second values (e.g., `[0.0361, 0.0126, ...]`), while `timeit` demonstrates that eliminating Python interpreter loop overhead drastically reduces runtime on large datasets[cite: 6].
* **E. Modified Real-World Case (Packet Jitter Variance Calculation):**
  * **Assumptions:** Network telemetry records `500,000` packet inter-arrival jitter samples in microseconds ($\mu\text{s}$), converted to milliseconds and squared to compute mean squared jitter error.
  * **Modified Input & Output:** `jitter_us = rng.uniform(100, 5000, 500_000)`; vectorized `(jitter_us / 1000.0) ** 2` computes all 500,000 squared jitter metrics in `< 0.003` s compared to `> 0.12` s via loop.

---

## Task 04: Floating-Point Arithmetic and Numerical Errors (`code/Task_04_numerical_errors.py`)[cite: 6, 7]
* **B. Inputs & Mathematical Operation:** Evaluates binary floating-point addition (`0.1 + 0.2` vs `0.3`), computes absolute error $|v_{\text{true}} - v_{\text{approx}}|$ and relative error for $\sqrt{2} \approx 1.414$, and compares direct $\frac{\sqrt{1+x}-1}{x}$ against rationalized $\frac{1}{\sqrt{1+x}+1}$ for $x \in \{10^{-4}, 10^{-8}, 10^{-12}\}$[cite: 7].
* **C. Parameter Change & Effect:** Decreasing $x$ to `1e-16` causes catastrophic cancellation in the direct formula: $\sqrt{1 + 10^{-16}}$ rounds to `1.0` in IEEE-754 float64, yielding `0.000000000000` (100% error), while the rationalized form accurately returns `0.500000000000`[cite: 7].
* **D. Output Interpretation:** `0.1 + 0.2 == 0.3` evaluates to `False` (`0.30000000000000004`) due to finite binary representation, proving why `np.isclose()` and algebraically stable formulations are mandatory in scientific code[cite: 7].
* **E. Modified Real-World Case (Financial Compound Interest Precision):**
  * **Assumptions:** Computing daily interest accrual $(1 + r)^n - 1$ for ultra-small per-second interest rates $r = 10^{-11}$ requires `np.expm1(n * np.log1p(r))` to prevent precision loss.
  * **Modified Input & Output:** For $r = 10^{-12}$, naive `(1 + r) - 1` suffers round-off noise, whereas `np.log1p(r)` preserves full 16-digit mantissa accuracy.

---

## Task 05: Matrix Algebra and Matrix Operations (`code/Task_05_matrix_algebra.py`)[cite: 8]
* **B. Inputs & Mathematical Operation:** Takes a $4 \times 2$ coordinate matrix of unit square vertices, constructs a $2 \times 2$ 2D rotation matrix ($45^\circ$) and a $2 \times 2$ non-uniform scaling matrix ($S_x = 2.0, S_y = 1.5$), multiplies them ($T = R \cdot S$), and transforms all vertices via $P' = P \cdot T^T$[cite: 8, 9].
* **C. Parameter Change & Effect:** Changing rotation angle from `45`° to `90`° rotates the scaled rectangle onto the negative x-axis (`[0, 0], [0, 2], [-1.5, 2], [-1.5, 0]`), while the determinant stays constant at `3.0` ($2.0 \times 1.5$)[cite: 8, 9].
* **D. Output Interpretation:** The transformation scales area by $\det(T) = 3.0$ (from $1\text{ unit}^2$ to $3\text{ units}^2$) and rotates every corner counterclockwise by $45^\circ$ around the origin[cite: 8, 9].
* **E. Modified Real-World Case (2D Drone Waypoint Coordinate Transformation):**
  * **Assumptions:** A quadcopter shifts from local sensor coordinates to world GPS grid coordinates via a $30^\circ$ heading alignment and $10\text{ m/unit}$ map scaling.
  * **Modified Input & Output:** Applying `angle = np.deg2rad(30)` and `scale = np.diag([10.0, 10.0])` maps local waypoint `[1.0, 0.5]` to `[6.16, 9.33]` meters in world coordinates.

---

## Task 06: Solution of Systems of Linear Equations (`code/Task_06_linear_systems.py`)[cite: 9]
* **B. Inputs & Mathematical Operation:** Solves the $2 \times 2$ linear system $Ax = b$ using `np.linalg.solve(A, b)` where $A = \begin{bmatrix} 4 & 2 \\ 8 & 12 \end{bmatrix}$ (CPU and RAM per server type) and $b = [20, 72]^T$ (total cluster resource consumption)[cite: 9, 10].
* **C. Parameter Change & Effect:** Increasing total observed memory in `b` from `72.0` GB to `88.0` GB (keeping CPU at `20.0`) shifts the solution from `(3 Type A, 4 Type B)` to `(2 Type A, 6 Type B)` servers[cite: 9, 10].
* **D. Output Interpretation:** Exactly `3` Type A servers and `4` Type B servers account for `20` CPU units and `72` GB RAM, confirmed by a residual norm of `0.0` ($Ax - b = [0, 0]$)[cite: 9, 10].
* **E. Modified Real-World Case (Cloud Bandwidth & Storage Allocation):**
  * **Assumptions:** Microservice Container X uses `2` Gbps bandwidth and `10` GB SSD; Container Y uses `5` Gbps bandwidth and `15` GB SSD; total usage is `34` Gbps and `120` GB.
  * **Modified Input & Output:** Solving $A = [[2, 5], [10, 15]]$, $b = [34, 120]$ yields `4.5` (or `4` X and `5` Y containers under integer provisioning) with zero residual error.

---

## Task 07: Gaussian Elimination and LU Decomposition (`code/Task_07_gaussian_lu.py`)[cite: 11]
* **B. Inputs & Mathematical Operation:** Factors a symmetric tridiagonal $3 \times 3$ matrix $A$ once via `scipy.linalg.lu_factor(A)` into $PA = LU$, then solves $Ax = b_i$ across three distinct right-hand side vectors $b_1, b_2, b_3$ using `lu_solve`[cite: 11].
* **C. Parameter Change & Effect:** Adding a fourth load vector `b_4 = np.array([20.0, 15.0, 12.0])` requires only $O(n^2)$ forward/back-substitution (`x = [6.439, 5.756, 5.919]`) without re-running the $O(n^3)$ matrix factorization[cite: 11].
* **D. Output Interpretation:** All three measurement sets produce exact solutions matching `np.linalg.solve` with residual norms around $10^{-15}$ (machine precision), demonstrating computational efficiency when system geometry stays constant[cite: 11, 12].
* **E. Modified Real-World Case (Multi-Scenario Structural Truss Load Analysis):**
  * **Assumptions:** A 3-joint bridge truss stiffness matrix $K$ remains fixed while testing three wind/traffic load vectors.
  * **Modified Input & Output:** Factoring stiffness matrix $K$ once with `lu_factor` computes joint displacements across all three load cases in a single pass with residual norms $< 10^{-15}$.

---

## Task 08: Iterative Methods for Linear Systems (`code/Task_08_iterative_linear.py`)[cite: 12]
* **B. Inputs & Mathematical Operation:** Solves a $5 \times 5$ strictly diagonally dominant tridiagonal steady-state heat conduction matrix $A$ with boundary vector $b = [100, 0, 0, 0, 20]^T$ using Jacobi and Gauss-Seidel iterative updates until $\|x^{(k+1)} - x^{(k)}\|_\infty < 10^{-8}$[cite: 12, 13].
* **C. Parameter Change & Effect:** Tightening tolerance `tol` from `1e-8` to `1e-12` increases required iterations for both solvers, while relaxing `tol` to `1e-3` halves iteration count at the cost of ~0.001 °C accuracy[cite: 13].
* **D. Output Interpretation:** Both methods converge to the exact linear temperature gradient `[86.67, 73.33, 60.00, 46.67, 33.33]` °C between the `100` °C left wall and `20` °C right wall[cite: 12, 13]. On the semilog plot, Gauss-Seidel's steeper downward slope proves it converges in roughly half as many iterations as Jacobi because it immediately uses updated components[cite: 13, 14].
* **E. Modified Real-World Case (2D Resistor Ladder Voltage Distribution):**
  * **Assumptions:** A 5-node resistive voltage divider connects a `12.0` V source on the left to a `3.3` V reference on the right.
  * **Modified Input & Output:** Setting `b = [12.0, 0., 0., 0., 3.3]` yields steady-state node voltages `[10.55, 9.10, 7.65, 6.20, 4.75]` V, with Gauss-Seidel converging in 28 iterations vs 54 for Jacobi.

---

## Task 09: Eigenvalues and Eigenvectors (`code/Task_09_eigenvalues.py`)[cite: 14]
* **B. Inputs & Mathematical Operation:** Computes eigenvalues and eigenvectors of a $4 \times 4$ undirected network adjacency matrix $A$ via `np.linalg.eig(A)`, extracts the dominant eigenvalue $\lambda_{\max}$, and normalizes its eigenvector to sum to 1[cite: 14, 15].
* **C. Parameter Change & Effect:** Adding a direct link between nodes A and D (`A[0, 3] = A[3, 0] = 1.0`) creates a complete graph $K_4$, increasing $\lambda_{\max}$ from `2.5616` to `3.0` and equalizing all node centrality scores to `0.2500`[cite: 14, 15].
* **D. Output Interpretation:** Nodes B and C have higher eigenvector centrality (`0.2808` each) than A and D (`0.2192` each) because B and C have degree 3 and connect to other well-connected nodes, verified by $\|Av - \lambda v\| \approx 0$[cite: 14, 15].
* **E. Modified Real-World Case (Microservice Dependency Hub Ranking):**
  * **Assumptions:** Four services (Auth, Gateway, Billing, Logging) form an internal API call graph; higher centrality indicates higher failure blast radius.
  * **Modified Input & Output:** Gateway and Auth achieve the highest centrality scores (`0.2808`), identifying them as primary single points of failure requiring redundancy.

---

## Task 10: Root Finding — Bisection and Fixed-Point Methods (`code/Task_10_bisection_fixed_point.py`)[cite: 15]
* **B. Inputs & Mathematical Operation:** Solves $f(T) = T - 25 - 10e^{-T/20} = 0$ on bracket $[20, 40]$ via `scipy.optimize.root_scalar(..., method="bisect")` and fixed-point iteration $T_{n+1} = 25 + 10e^{-T_n/20}$ from $T_0 = 30.0$ to tolerance $10^{-10}$[cite: 15, 16].
* **C. Parameter Change & Effect:** Changing ambient base temperature from `25` to `35` shifts the equilibrium root from `27.502216` °C to `36.609142` °C and requires expanding the bisection bracket to `[30, 45]`[cite: 16].
* **D. Output Interpretation:** Both methods converge to `27.502216` °C (fixed-point takes 14 iterations)[cite: 16]. The plot shows the monotonically increasing curve $f(T)$ crossing the horizontal zero-axis cleanly at the dashed vertical equilibrium line[cite: 16].
* **E. Modified Real-World Case (Server CPU Thermal Equilibrium):**
  * **Assumptions:** CPU steady-state temperature satisfies $T - 30 - 18e^{-T/25} = 0$ under constant workload and active cooling.
  * **Modified Input & Output:** Bisection on `[25, 50]` locates the thermal steady-state root at `34.478102` °C, safely below the `85` °C throttle limit.

---

## Task 11: Root Finding — Newton-Raphson and Secant Methods (`code/Task_11_newton_secant.py`)[cite: 17]
* **B. Inputs & Mathematical Operation:** Computes Internal Rate of Return (IRR) where $\text{NPV}(r) = \sum_{t=0}^4 \frac{CF_t}{(1+r)^t} = 0$ for `cash_flows = [-10000, 3000, 3500, 4000, 2500]`, comparing analytical-derivative Newton-Raphson ($x_0 = 0.10$) against derivative-free Secant ($x_0 = 0.05, x_1 = 0.20$)[cite: 17, 18].
* **C. Parameter Change & Effect:** Increasing Year 4 cash inflow from `2500` to `4500` increases project IRR from `11.797253%` to `17.345812%`[cite: 17, 18].
* **D. Output Interpretation:** Both Newton-Raphson and Secant converge to `11.797253%` where $\text{NPV} = 0.000000$, meaning the project is profitable whenever the organization's cost of capital is below `11.80%`[cite: 17, 18].
* **E. Modified Real-World Case (SaaS Infrastructure Upgrade Break-Even Rate):**
  * **Assumptions:** Upfront cloud migration cost of `-15000` USD generates annual savings `[4500, 5200, 6000, 6500]` over 4 years.
  * **Modified Input & Output:** Newton-Raphson converges in 5 iterations to an IRR of `16.489120%`.

---

## Task 12: Polynomial and Nonlinear Equation Solving (`code/Task_12_nonlinear_equations.py`)[cite: 18]
* **B. Inputs & Mathematical Operation:** Uses `scipy.optimize.root` (Powell hybrid solver) to solve the $2 \times 2$ nonlinear system $x^2 + y^2 - 4 = 0$ (circle of radius 2) and $y - (x^2 - 1) = 0$ (upward parabola) from initial guesses `[1.0, 1.0]` and `[-1.0, 1.0]`[cite: 18, 19].
* **C. Parameter Change & Effect:** Increasing the circle radius squared from `4` to `9` ($x^2 + y^2 = 9$) pushes the intersection points outward from $(\pm 1.5175, 1.3028)$ to $(\pm 1.8323, 2.3574)$[cite: 18, 19].
* **D. Output Interpretation:** Starting from symmetric guesses locates both symmetric intersection points `(1.517490, 1.302776)` and `(-1.517490, 1.302776)` with near-zero residuals ($\sim 10^{-14}$), showing how initial guesses select distinct roots in nonlinear systems[cite: 18, 19].
* **E. Modified Real-World Case (Robot Arm Reach & Safety Boundary Intersection):**
  * **Assumptions:** A robotic arm end-effector boundary $x^2 + y^2 = 2.25$ intersects a parabolic conveyor path $y = 0.5x^2 - 0.5$.
  * **Modified Input & Output:** Solving from `[1.0, 0.5]` and `[-1.0, 0.5]` identifies the exact entry/exit pickup coordinates at `(\pm 1.4386, 0.5348)` m.

---

## Task 13: Interpolation (`code/Task_13_interpolation.py`)[cite: 19]
* **B. Inputs & Mathematical Operation:** Takes 7 hourly temperature points from `08:00` to `14:00` and constructs piecewise linear (`interp1d(..., kind="linear")`) and $C^2$-continuous cubic spline (`CubicSpline`) interpolants evaluated at 10-minute intervals (`1/6` hour)[cite: 19, 20].
* **C. Parameter Change & Effect:** Lowering the `11:00` reading from `24.0` to `21.0` °C causes the linear interpolant to form sharp corners at `10:00` and `11:00`, while `CubicSpline` smoothly bends across adjacent intervals, shifting the `10:30` estimate from `23.33` °C to `21.74` °C[cite: 19, 20].
* **D. Output Interpretation:** At `10:30`, linear interpolation averages adjacent points (`23.25` °C) while cubic spline accounts for curvature (`23.33` °C)[cite: 20]. On the graph, the cubic spline forms a smooth natural curve through every scatter point whereas the linear curve shows visible kinks at each hourly node[cite: 20].
* **E. Modified Real-World Case (Battery Discharge Curve Resampling):**
  * **Assumptions:** Voltage readings logged every 15 minutes `[4.20, 4.05, 3.92, 3.81, 3.70]` V are resampled to 1-minute resolution for state-of-charge estimation.
  * **Modified Input & Output:** `CubicSpline` provides smooth per-minute voltage estimates without step discontinuities at the 15-minute measurement boundaries.

---

## Task 14: Curve Fitting and Approximation (`code/Task_14_curve_fitting.py`)[cite: 20]
* **B. Inputs & Mathematical Operation:** Fits a quadratic polynomial $y = ax^2 + bx + c$ to 7 pairs of `(users, response_ms)` via nonlinear least squares (`scipy.optimize.curve_fit`), computes RMSE, and extrapolates response time at `400` concurrent users[cite: 20, 21].
* **C. Parameter Change & Effect:** Changing the model from quadratic to linear ($y = bx + c$) increases RMSE from `0.62` ms to `16.45` ms and severely underpredicts latency at `400` users (`326` ms vs `382.86` ms), proving the nonlinear queueing delay[cite: 21].
* **D. Output Interpretation:** The fitted model $y = 0.001786x^2 + 0.0286x + 76.14$ fits the measurements with an RMSE of `0.62` ms[cite: 21]. The upward-curving plot illustrates superlinear latency growth as server saturation approaches `400` users (`382.86` ms)[cite: 21].
* **E. Modified Real-World Case (Database Query Latency vs Index Size):**
  * **Assumptions:** Measuring query execution time (ms) across table sizes `[10k, 50k, 100k, 250k, 500k]` rows using a logarithmic model $y = a \ln(x) + b$.
  * **Modified Input & Output:** `curve_fit` confirms $O(\log n)$ B-tree index scaling with RMSE `< 0.4` ms.

---

## Task 15: Numerical Differentiation (`code/Task_15_numerical_differentiation.py`)[cite: 22]
* **B. Inputs & Mathematical Operation:** Estimates instantaneous speed $v(t) = dx/dt$ from 7 GPS position samples (`t = 0..6` s) using forward difference at $t=0$, second-order central differences $\frac{x_{i+1}-x_{i-1}}{2\Delta t}$ for interior points $t=1..5$, and backward difference at $t=6$[cite: 22].
* **C. Parameter Change & Effect:** Introducing a `0.5` m GPS noise spike at $t=3$ (`16.6` m instead of `16.1` m) leaves $v(3)$ unchanged (since central difference uses $x_4$ and $x_2$) but perturbs $v(2)$ and $v(4)$ by $\pm 0.25$ m/s, illustrating noise amplification in numerical derivatives[cite: 22].
* **D. Output Interpretation:** Estimated speed rises steadily from `4.80` m/s at $t=0$ to `8.00` m/s at $t=6$, producing a nearly straight upward line on the graph that indicates constant vehicle acceleration of ~`0.64` $\text{m/s}^2$[cite: 22].
* **E. Modified Real-World Case (Network Throughput Rate-of-Change Monitoring):**
  * **Assumptions:** Cumulative bytes transferred (`MB`) sampled every `5` seconds are differentiated via central differences to compute instantaneous bandwidth (`MB/s`).
  * **Modified Input & Output:** Differentiating cumulative transfer `[0, 12, 30, 55, 82]` MB yields peak instantaneous throughput of `5.2` MB/s during the burst window.

---

## Task 16: Numerical Integration (`code/Task_16_numerical_integration.py`)[cite: 23]
* **B. Inputs & Mathematical Operation:** Integrates 7 smart-meter power readings (`power_kw`) recorded at 2-hour intervals over 12 hours using both the trapezoidal rule (`np.trapezoid`) and Simpson's parabolic rule (`scipy.integrate.simpson`)[cite: 23].
* **C. Parameter Change & Effect:** Increasing peak midday power at `6` hours from `1.8` kW to `2.4` kW increases trapezoidal energy by `1.200` kWh (`15.700` to `16.900` kWh) and Simpson's energy by `1.600` kWh (`15.800` to `17.400` kWh) due to Simpson's `4/3` weight on interior odd-indexed nodes[cite: 23].
* **D. Output Interpretation:** Total energy consumed over the 12-hour window is `15.700` kWh (trapezoidal) and `15.800` kWh (Simpson's); the `0.100` kWh difference arises because Simpson's rule captures the smooth parabolic peak at midday better than straight-line segments[cite: 23].
* **E. Modified Real-World Case (Solar Panel Daily Energy Yield):**
  * **Assumptions:** Inverter output power (`kW`) logged every 2 hours from 06:00 to 18:00 `[0.0, 1.2, 3.1, 4.2, 3.4, 1.5, 0.0]` kW.
  * **Modified Input & Output:** `simpson()` computes `27.067` kWh total solar generation over the 12-hour daylight window.

---

## Task 17: Ordinary Differential Equations — Euler Method (`code/Task_17_euler_ode.py`)[cite: 24]
* **B. Inputs & Mathematical Operation:** Numerically integrates Newton's law of cooling $\frac{dT}{dt} = -k(T - T_{\text{ambient}})$ from $T(0) = 90.0$ °C to $t = 30$ min ($k = 0.12$, $T_{\text{ambient}} = 25.0$ °C) using explicit forward Euler $T_{i+1} = T_i + h f(T_i)$ with step $h = 1.0$ min, validated against $T(t) = 25 + 65e^{-0.12t}$[cite: 24, 25].
* **C. Parameter Change & Effect:** Reducing step size $h$ from `1.0` min to `0.1` min reduces the final absolute error at $t = 30$ min by roughly 10x (from `0.1149` °C down to `0.0110` °C), confirming Euler's first-order $O(h)$ global truncation error[cite: 24, 25].
* **D. Output Interpretation:** After 30 minutes, Euler predicts `26.66` °C vs the exact `26.78` °C (`0.1149` °C error)[cite: 25]. The plot shows both curves decaying exponentially toward the `25` °C room temperature, with Euler slightly underestimating the exact curve because it uses the steeper slope at the start of each step[cite: 24, 25].
* **E. Modified Real-World Case (Server Room Cooling After AC Failure/Recovery):**
  * **Assumptions:** A hot server rack at `65.0` °C cools toward a `20.0` °C CRAC setpoint with $k = 0.15\text{ min}^{-1}$ and $h = 0.5$ min.
  * **Modified Input & Output:** Euler simulation shows the rack drops below `25.0` °C within `14.5` minutes.

---

## Task 18: Numerical Solution of ODEs Using SciPy (`code/Task_18_scipy_ode.py`)[cite: 25]
* **B. Inputs & Mathematical Operation:** Integrates the RC charging ODE $\frac{dV}{dt} = \frac{V_s - V}{RC}$ ($V_s = 5.0$ V, $R = 1000\ \Omega$, $C = 0.001$ F, $V_0 = 0.0$ V) over $t \in [0, 5]$ s using adaptive Runge-Kutta (`scipy.integrate.solve_ivp`) with `rtol=1e-9, atol=1e-11`[cite: 25, 26].
* **C. Parameter Change & Effect:** Doubling capacitance $C$ from `0.001` F to `0.002` F doubles the time constant $\tau = RC$ from `1.0` s to `2.0` s, reducing capacitor voltage at $t = 5$ s from `4.9663` V (99.3% charged) to `4.5896` V (91.8% charged)[cite: 26].
* **D. Output Interpretation:** At $t = 5$ s ($5\tau$), capacitor voltage reaches `4.9663` V with a maximum error of $\sim 10^{-10}$ V against the analytical curve[cite: 26]. On the graph, the `solve_ivp` curve overlaps the dashed exact exponential curve identically[cite: 26].
* **E. Modified Real-World Case (SIR Epidemic Spread Model):**
  * **Assumptions:** Modeling susceptible, infected, and recovered populations using a coupled 3-ODE system passed to `solve_ivp`.
  * **Modified Input & Output:** `solve_ivp` tracks peak infection timing and herd-immunity stabilization accurately with adaptive step control.

---

## Task 19: Numerical Optimization (`code/Task_19_optimization.py`)[cite: 27]
* **B. Inputs & Mathematical Operation:** Minimizes linear server cost $f(x, y) = 12x + 18y$ subject to inequality capacity constraint $100x + 170y \ge 1000$ and bounds $x, y \ge 0$ using Sequential Least Squares Programming (`scipy.optimize.minimize(..., method="SLSQP")`), followed by an exhaustive integer grid search[cite: 27, 28].
* **C. Parameter Change & Effect:** Increasing required minimum capacity from `1000` to `1500` shifts the continuous optimum from `(0.000, 5.882)` (cost `105.88`) to `(0.000, 8.824)` (cost `158.82`), and the best integer mix becomes `(0, 9)` at cost `162` (`1530` capacity)[cite: 27, 28].
* **D. Output Interpretation:** Type B servers offer better cost-per-capacity (`18/170 = 0.1059` vs `12/100 = 0.1200`), so SLSQP allocates `0` Type A and `5.882` Type B units (`105.88` cost); when restricted to whole servers, `(0 Type A, 6 Type B)` achieves `1020` capacity at minimum integer cost `108`[cite: 27, 28].
* **E. Modified Real-World Case (GPU Cluster Provisioning Under Budget & VRAM Constraints):**
  * **Assumptions:** Choosing counts of GPU Node A (`$15/hr`, `24 GB` VRAM) and GPU Node B (`$28/hr`, `48 GB` VRAM) to supply at least `240 GB` total VRAM.
  * **Modified Input & Output:** Optimization selects `5` units of Node B (`240 GB` VRAM) at `$140/hr` rather than `10` units of Node A (`$150/hr`).

---

## Task 20: Random Numbers and Monte Carlo Methods (`code/Task_20_monte_carlo.py`)[cite: 29]
* **B. Inputs & Mathematical Operation:** Draws $N = 100,000$ normally distributed demand samples $D \sim \mathcal{N}(\mu=850, \sigma=120)$ using `rng.normal` and estimates overload probability $P(D > 1000)$ along with cumulative running probabilities on a logarithmic scale[cite: 29, 30].
* **C. Parameter Change & Effect:** Upgrading server capacity from `1000.0` to `1100.0` pushes the threshold from $1.25\sigma$ to $2.08\sigma$ above the mean, dropping overload probability from `10.5650%` (~10,565 overloads) down to `1.8620%` (~1,862 overloads)[cite: 29].
* **D. Output Interpretation:** There is a `10.5650%` risk of exceeding the `1000` unit capacity[cite: 29]. The semilog convergence plot shows wild statistical fluctuations when $N < 1,000$ that flatten out smoothly around `0.1056` by $N = 100,000$, illustrating the Law of Large Numbers ($O(1/\sqrt{N})$ convergence)[cite: 29, 30].
* **E. Modified Real-World Case (Project Completion Deadline Risk Simulation):**
  * **Assumptions:** Software sprint duration follows $\mathcal{N}(\mu=12.0, \sigma=2.5)$ days against a hard client deadline of `16.0` days across `100,000` trials.
  * **Modified Input & Output:** Monte Carlo simulation estimates a `5.48%` probability of missing the 16-day release deadline.