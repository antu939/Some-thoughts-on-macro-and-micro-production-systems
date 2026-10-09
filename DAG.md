# Supply Chain Network Architecture

## 1. Overall Structure

The supply chain network is represented as a directed acyclic graph (DAG) with static physical attributes that can operate along a unified timeline. The overall architecture consists of four layers:

$$
\boxed{
\text{Single-source production DAG}
\longrightarrow
\text{Multi-source production DAG}
\longrightarrow
\text{DAG with embedded static physical attributes}
\longrightarrow
\text{Time-evolving dynamical system}
}
$$

Only two kinds of nodes are retained in the network:

$$
\boxed{
M=\text{Material-State-at-Location / Available material inventory state}
}
$$

$$
\boxed{
P=\text{Production Process}
}
$$

$M$ denotes an independent stock of available material of a given type, at a given location and in a given state, which can accumulate and wait to be used. $P$ denotes a production process that transforms materials.

All directed edges involve transportation. Whenever material moves along an edge from an upstream object to a downstream object, that edge stores a transportation capacity and transportation delay. Edges connected to a production process $P$ also specify quantitative input or output relationships:

$$
\boxed{
M\rightarrow P:
\text{Transportation}+\text{Input coefficient}
}
$$

$$
\boxed{
P\rightarrow M:
\text{Output coefficient}+\text{Transportation}
}
$$

$$
\boxed{
M\rightarrow M:
\text{Pure transportation}
}
$$

When the same output of a $P$ needs to supply multiple downstream production processes, the corresponding $P\rightarrow M$ branch edges also store allocation ratios describing how that output is distributed among downstream branches.

The current core scope covers network structure, static physical attributes, and dynamics. Network-generation parameters controlling network structure, economic policies, procurement strategies, optimization, and recovery decisions are excluded from the core DAG for now.

---

## 2. Single-Source Production Network

The most basic production unit is:

$$
\boxed{
M^{in}\rightarrow P\rightarrow M^{out}
}
$$

Here $M^{in}$ is the available input inventory before the production process, and $M^{out}$ is the available output inventory reached after production output traverses its corresponding transportation edge.

When two production processes are connected in sequence, the standard structure is fixed as:

$$
\boxed{
P_i
\rightarrow
M_{ij}^{(1)}
\rightarrow
M_{ij}^{(2)}
\rightarrow
P_j
}
$$

Including the production units before and after gives:

$$
\boxed{
M_i^{in}
\rightarrow
P_i
\rightarrow
M_{ij}^{(1)}
\rightarrow
M_{ij}^{(2)}
\rightarrow
P_j
\rightarrow
M_j^{out}
}
$$

Two independent $M$ nodes are retained between adjacent production processes.

The first inventory:

$$
M_{ij}^{(1)}
$$

Represents post-production inventory that arrives after upstream $P_i$ completes production and the output traverses the $P_i\rightarrow M_{ij}^{(1)}$ transportation edge.

The second inventory:

$$
M_{ij}^{(2)}
$$

Represents the available inventory after intermediate logistics, immediately before the downstream production process.

Thus, the local physical process is:

$$
\boxed{
P_i
\xrightarrow{\text{output transport}}
M_{ij}^{(1)}
\xrightarrow{\text{pure transport}}
M_{ij}^{(2)}
\xrightarrow{\text{input transport}}
P_j
}
$$

All three edges have transportation capacities and transportation delays.

Even if real-world logistics involve trucks, railways, ports, ocean shipping, customs clearance, distribution centers, or other intermediate steps, no additional $M$ nodes are introduced. These internal logistics details are folded into the transportation parameters of the existing edges. In particular, the edge between the two inventories:

$$
M_{ij}^{(1)}\rightarrow M_{ij}^{(2)}
$$

Represents the complete intermediate logistics process between the two inventory locations.

The lengths of edges in the drawing can help convey spatial facts. Drawing a production node close to an inventory node can indicate that they lie in the same facility or nearby; drawing two $M$ nodes close together can indicate that two real inventory facilities are geographically near; cross-regional logistics may be drawn with longer edges. Graphical edge length offers spatial intuition, but the actual transportation times used in the dynamics are determined by each edge's own $\tau_e^{trans}$.

One production process may require multiple inputs simultaneously. For example:

$$
M_A\rightarrow P,\qquad
M_B\rightarrow P,\qquad
M_C\rightarrow P
$$

It may also produce multiple outputs:

$$
P\rightarrow M_D,\qquad
P\rightarrow M_E
$$

The quantitative input and output relationships are determined, respectively, by the input and output coefficients on the edges.

Single-source means that a required component has only one actual source of supply in the current network; it does not mean a production process can have only one input or output.

---

## 3. Multiple Sources, Branches, and Allocation

A multi-source network retains the same $P$, $M$, and edge semantics, without introducing new node types.

### Multiple Upstream Production Processes Supplying One Downstream Process

If both $P_1$ and $P_2$ supply materials needed by the same downstream process $P_D$, they can be represented as two independent source paths:

$$
P_1
\rightarrow
M_{1D}^{(1)}
\rightarrow
M_{1D}^{(2)}
\rightarrow
P_D
$$

$$
P_2
\rightarrow
M_{2D}^{(1)}
\rightarrow
M_{2D}^{(2)}
\rightarrow
P_D
$$

Each path has its own inventory IDs, inventory capacities, and transportation parameters. When multiple upstream processes supply a single $P_D$, no additional ratio specifying "how much to allocate to $P_D$" is needed on upstream $P\rightarrow M$ edges; the downstream process's demand for each input is directly defined by the input coefficients on $M\rightarrow P$ edges.

For example:

$$
\boxed{
2A+3B\rightarrow C
}
$$

This corresponds to:

$$
a_{A,p}=2,\qquad
a_{B,p}=3
$$

And:

$$
b_{p,C}=1
$$

This means that each unit of operation of production process $P$ consumes $2$ units of $A$ and $3$ units of $B$, and produces $1$ unit of $C$.

If $A$ and $B$ come from different upstream production processes, their source paths remain independent; the quantities ultimately entering $P$ continue to be governed by:

$$
2:3
$$

This input recipe.

### One Production Process Supplying Multiple Downstream Processes

If the same output from production process $P_i$ must supply multiple downstream processes $P_1,\ldots,P_k$, each corresponding $P_i\rightarrow M$ branch edge stores an allocation ratio in addition to its output coefficient and transportation attributes:

$$
\boxed{
\rho_{ie}
}
$$

For example:

$$
P_i
\rightarrow
M_{i1}^{(1)}
\rightarrow
M_{i1}^{(2)}
\rightarrow
P_1
$$

$$
P_i
\rightarrow
M_{i2}^{(1)}
\rightarrow
M_{i2}^{(2)}
\rightarrow
P_2
$$

If the two paths distribute the same output material from $P_i$, then:

$$
\rho_{i1}\ge 0,\qquad
\rho_{i2}\ge 0
$$

And they normally satisfy:

$$
\boxed{
\rho_{i1}+\rho_{i2}=1
}
$$

More generally, for the set $\mathcal B(p,m)$ of all downstream branches for the same output material:

$$
\boxed{
\sum_{e\in\mathcal B(p,m)}\rho_e=1
}
$$

If $P_i$ produces $b_{pm}$ units of material $m$ per unit of operation, the nominal output quantity allocated to branch $e$ is:

$$
\boxed{
\rho_e\,b_{pm}
}
$$

An allocation ratio divides the same output among multiple downstream branches. If $P$ produces different types of joint products, their quantities are first defined by their respective output coefficients $b_{pm}$; allocation ratios are used only when one particular output is itself split among multiple downstream branches.

In the current static network, $\rho_e$ is stored as a given allocation ratio. Subsequent work on scheduling or active control could extend it to a time-varying $\rho_e(t)$.

### Inventory Instances Are Not Automatically Merged

Each $M$ is a distinct inventory instance with a unique ID:

$$
\boxed{
id_m
}
$$

Even if two inventories have exactly the same:

$$
(material,\ state,\ location)
$$

They may still be represented as:

$$
M^{(1)},\qquad M^{(2)}
$$

And each has its own:

$$
q_{m_1}(t),\qquad q_{m_2}(t)
$$

As well as its own inventory capacity, upstream and downstream connections, and shock history.

Thus, $material$, $state$, and $location$ describe an inventory's physical attributes, whereas $id_m$ identifies the inventory instance. The same $M$ node is used only if the inventories are in fact the same physical inventory instance.

The same product can correspond to multiple production DAGs. If two production arrangements remain largely independent and only weakly coupled over time, they may be modeled as separate networks:

$$
\boxed{
\text{Product identity}
\neq
\text{Production-network identity}
}
$$

---

## 4. Static Physical Attributes

The static layer stores the essential physical information needed for the production network to operate correctly.

The current static network is written as:

$$
\boxed{
G^S=
\left(
M,\,
P,\,
E^{MP},\,
E^{PM},\,
E^{MM};
\Theta
\right)
}
$$

Where:

$$
E^{MP}\subseteq M\times P
$$

Denotes input transportation edges from inventories to production processes;

$$
E^{PM}\subseteq P\times M
$$

Denotes output transportation edges from production processes to inventories;

$$
E^{MM}\subseteq M\times M
$$

Denotes pure transportation edges between inventories.

All edges together form:

$$
E=E^{MP}\cup E^{PM}\cup E^{MM}
$$

And all share transportation-capacity and transportation-delay attributes.

### Material Node $M$

A Material node stores at least:

$$
\boxed{
M_m:
\left(
id_m,\,
material_m,\,
state_m,\,
location_m,\,
\bar q_m
\right)
}
$$

Where:

$$
id_m=\text{Unique inventory instance ID}
$$

$$
material_m=\text{Material type}
$$

$$
state_m=\text{Material state}
$$

$$
location_m=\text{Material location}
$$

$$
\bar q_m=\text{Maximum allowable stock of this inventory instance}
$$

The actual inventory quantity:

$$
\boxed{
q_m(t)
}
$$

Is a dynamic state and satisfies:

$$
\boxed{
0\le q_m(t)\le \bar q_m
}
$$

The initial inventory is:

$$
\boxed{
q_m(0)
}
$$

If necessary, $M$ may also store additional static information such as the owning firm, storage type, or facility ID; these fields do not change the basic physical meaning of an inventory node.

### Production Node $P$

$P$ denotes a production process. A production node stores at least:

$$
\boxed{
P_p:
\left(
id_p,\,
process_p,\,
location_p,\,
\bar c_p,\,
\tau_p^{prod}
\right)
}
$$

Where:

$$
id_p=\text{Production-process instance ID}
$$

$$
process_p=\text{Production process or activity}
$$

$$
location_p=\text{Production location}
$$

$$
\bar c_p=\text{Maximum production capacity per unit time / rated output rate}
$$

$$
\tau_p^{prod}=\text{Production delay}
$$

The actual flow rate during production is denoted by:

$$
\boxed{
u_p(t)
}
$$

And satisfies:

$$
\boxed{
0\le u_p(t)\le \bar c_p
}
$$

Production capacity determines how much production activity can be completed per unit time; production delay determines how long it takes for a batch of inputs to become output after entering production.

Multiple $P$ nodes within the same company may share firm-level total-output limits, equipment limits, or other joint constraints. The current core DAG does not explicitly represent such firm-level aggregate-output constraints; each $P$ stores only its own local production parameters.

### Input Edge $M\rightarrow P$

Any input edge:

$$
e=(m,p)\in E^{MP}
$$

Stores:

$$
\boxed{
e^{MP}:
\left(
a_{mp},\,
\bar c_e^{trans},\,
\tau_e^{trans}
\right)
}
$$

Where:

$$
a_{mp}=\text{Input coefficient}
$$

This specifies how many units of input material from inventory $M_m$ are required for one unit of operation of production process $P_p$.

Also:

$$
\bar c_e^{trans}=\text{Maximum transportation capacity of this edge per unit time}
$$

$$
\tau_e^{trans}=\text{Transportation delay of this edge}
$$

Thus $M\rightarrow P$ expresses both "delivery into production" and "the quantity needed to complete one unit of production."

For:

$$
2A+3B\rightarrow C
$$

We have:

$$
\boxed{
a_{A,p}=2,\qquad a_{B,p}=3
}
$$

### Output Edge $P\rightarrow M$

Any output edge:

$$
e=(p,m)\in E^{PM}
$$

Stores at least:

$$
\boxed{
e^{PM}:
\left(
b_{pm},\,
\bar c_e^{trans},\,
\tau_e^{trans}
\right)
}
$$

Where:

$$
b_{pm}=\text{Output coefficient}
$$

This specifies how many units of output material destined for inventory $M_m$ are produced per unit of operation of production process $P_p$.

For:

$$
2A+3B\rightarrow C
$$

We have:

$$
\boxed{
b_{p,C}=1
}
$$

If the same production output from $P_p$ is distributed among several downstream branches, the corresponding $P\rightarrow M$ branch edges gain an allocation ratio:

$$
\boxed{
e^{PM}_{branch}:
\left(
b_{pm},\,
\rho_e,\,
\bar c_e^{trans},\,
\tau_e^{trans}
\right)
}
$$

Here $\rho_e$ describes the share of that output allocated to the current branch.

Thus, a $P\rightarrow M$ branch carries three kinds of physical information:

$$
\boxed{
\text{Production output relation}
+
\text{Allocation}
+
\text{Transportation}
}
$$

When no one-to-many distribution occurs, $\rho_e$ can be omitted.

### Inventory Edge $M\rightarrow M$

Any transportation edge between inventories:

$$
e=(m_i,m_j)\in E^{MM}
$$

Stores:

$$
\boxed{
e^{MM}:
\left(
\bar c_e^{trans},\,
\tau_e^{trans}
\right)
}
$$

It represents pure logistics movement between two distinct inventory instances, without an input or output coefficient.

If the model needs to record physical distances explicitly, the following can also be added to all edges:

$$
d_e
$$

Giving:

$$
\boxed{
e:
\left(
\bar c_e^{trans},\,
\tau_e^{trans},\,
d_e
\right)
}
$$

$d_e$ represents actual spatial scale and assists graph layout; propagation times in the dynamics are still specified directly by $\tau_e^{trans}$.

The minimum static physical information can currently be summarized as:

$$
\boxed{
\begin{array}{lll}
M
&:&
(id_m,\ material,\ state,\ location,\ \bar q_m)
\\[2mm]
P
&:&
(id_p,\ process,\ location,\ \bar c_p,\ \tau_p^{prod})
\\[2mm]
M\rightarrow P
&:&
(a_{mp},\ \bar c_e^{trans},\ \tau_e^{trans})
\\[2mm]
P\rightarrow M
&:&
(b_{pm},\ [\rho_e],\ \bar c_e^{trans},\ \tau_e^{trans})
\\[2mm]
M\rightarrow M
&:&
(\bar c_e^{trans},\ \tau_e^{trans})
\end{array}
}
$$

Here $[\rho_e]$ is used only when the same output from one $P$ is allocated among multiple downstream branches.

---

## 5. Capacity, Delay, and In-Transit States

Capacity and delay describe different physical constraints.

Production capacity:

$$
\boxed{
\bar c_p
}
$$

Determines the maximum amount of production activity that can occur per unit time.

Production delay:

$$
\boxed{
\tau_p^{prod}
}
$$

Determines how long it takes for inputs entering $P$ to become production outputs.

Every edge has a transportation capacity:

$$
\boxed{
\bar c_e^{trans}
}
$$

Determines how much material can move along the edge per unit time at most.

Every edge also has a transportation delay:

$$
\boxed{
\tau_e^{trans}
}
$$

Determines how long material takes to reach the other end after entering that edge.

Therefore:

$$
\boxed{ \text{Capacity} = \text{Maximum amount processed or transported per unit time} }
$$

$$
\boxed{ \text{Delay} = \text{Time for material in a process to reach its next state} }
$$

For a segment:

$$
M_A\rightarrow P\rightarrow M_B
$$

Suppose:

$$
\tau_{A,P}^{trans}=1h,\qquad
\tau_p^{prod}=3h,\qquad
\tau_{P,B}^{trans}=2h
$$

Then the minimum physical time for a batch to leave $M_A$ and finally reach $M_B$ is:

$$
\boxed{
1h+3h+2h=6h
}
$$

Before transportation is complete, material belongs to the corresponding edge's in-transit state and is not counted in the destination inventory.

Therefore, material in an operating network may be:

$$
\boxed{
\text{In inventory}
+
\text{In transit}
+
\text{In production}
}
$$

These are the three basic physical states.

---

## 6. Dynamical System

Once the static network is determined, a unified timeline is introduced:

$$
\boxed{
t:0\rightarrow T
}
$$

Along with an initial state:

$$
\boxed{
S_0
}
$$

The most basic initial state includes:

$$
q_m(0)
$$

Initial work-in-progress (WIP) and initial in-transit material may also be included when needed.

As time advances, the system produces a dynamic state:

$$
\boxed{
S(t)
}
$$

### Inventory

Each inventory instance has an independent inventory trajectory:

$$
\boxed{
q_m(t)
}
$$

Satisfying:

$$
\boxed{
0\le q_m(t)\le\bar q_m
}
$$

Inventory increases only when transportation actually arrives at $M$. When material departs $M$ via any outgoing edge, the corresponding amount is removed from the currently available inventory.

Let $In(m)$ and $Out(m)$ denote the incoming and outgoing edge sets of inventory node $m$, respectively. Under a continuous-flow approximation with fixed transportation delays and lossless transportation:

$$
\boxed{ \dot q_m(t) = \sum_{e\in In(m)} f_e\!\left(t-\tau_e^{trans}\right) - \sum_{e\in Out(m)} f_e(t) }
$$

Here $f_e(t)$ is the actual transportation flow rate entering edge $e$ at time $t$.

### Transportation Flows and In-Transit Material

The actual transportation flow rate on any edge is:

$$
\boxed{
f_e(t)
}
$$

Satisfying:

$$
\boxed{
0\le f_e(t)\le\bar c_e^{trans}
}
$$

Material enters edge $e$ at time $t$ and arrives at the downstream node at:

$$
t+\tau_e^{trans}
$$

Thus reaching the downstream node.

If transportation is lossless, the amount in transit on a fixed-delay edge can be written as:

$$
\boxed{ z_e(t) = \int_{t-\tau_e^{trans}}^{t}f_e(s)\,ds }
$$

Thus $P\rightarrow M$, $M\rightarrow M$, and $M\rightarrow P$ may each have independent in-transit states.

### Production Flow Rates and Work-in-Progress

The actual operating intensity of a production process is:

$$
\boxed{
u_p(t)
}
$$

And satisfies:

$$
\boxed{
0\le u_p(t)\le\bar c_p
}
$$

For an input edge $M_m\rightarrow P_p$, each unit of operation of $P_p$ requires:

$$
a_{mp}
$$

Units of input. Multi-input production must simultaneously satisfy the relevant input recipe.

For example:

$$
2A+3B\rightarrow C
$$

If the planned operating intensity during a certain period is:

$$
u_p(t)=10
$$

Then the required inputs, which must have completed input transportation and be available to the process, are:

$$
20
$$

Units of $A$, and:

$$
30
$$

Units of $B$.

Once material actually enters production, it remains in the WIP / in-production state until:

$$
\tau_p^{prod}
$$

Has elapsed:

$$
\boxed{
\text{production input at }t
\Rightarrow
\text{production output at }t+\tau_p^{prod}
}
$$

The output coefficients determine the quantities after production is completed.

If:

$$
b_{p,C}=1
$$

Then $10$ units of production activity produce:

$$
10C
$$

These outputs then enter the corresponding $P\rightarrow M$ transportation edges.

### Dynamic Meaning of One-to-Many Allocation

When the same output supplies multiple downstream processes through several $P\rightarrow M$ branches, static allocation ratios determine each branch's nominal share.

Let the output coefficient of production process $P_p$ for material $C$ be:

$$
b_{p,C}
$$

Let the allocation ratios of two branches be:

$$
\rho_1,\qquad\rho_2
$$

With:

$$
\rho_1+\rho_2=1
$$

Then the nominal release quantities to the two branches after production is completed are:

$$
\boxed{
\rho_1 b_{p,C}u_p(t-\tau_p^{prod})
}
$$

And:

$$
\boxed{
\rho_2 b_{p,C}u_p(t-\tau_p^{prod})
}
$$

Each branch is subsequently subject to the transportation capacity and delay of its own $P\rightarrow M$ edge.

If a branch's transportation capacity, destination inventory capacity, or other physical conditions cannot accommodate the assigned allocation, the feasible production flow rate is constrained accordingly. Future active scheduling could change allocation by adjusting $\rho_e(t)$.

---

## 7. Dynamic History and Shock Propagation

Running the static network under the dynamical rules produces a trajectory / event history, including:

- The time-varying $q_m(t)$ of each inventory instance;
- The actual $u_p(t)$ of each production process;
- The actual $f_e(t)$ of each edge;
- Production start and completion times for each material batch;
- Amounts of in-transit material on each edge;
- Actual allocation outcomes for one-to-many branches;
- Changes in production capacity after shocks;
- Shock propagation paths;
- Recovery processes.

The overall relationship is:

$$
\boxed{
\text{Static Network}
+
\text{Dynamics}
\longrightarrow
\text{Trajectory / Dynamic History}
}
$$

Dynamic history is an operational result, not part of the static topology.

Shocks can enter the system through changes in production capacities or the transportation capacity of any edge. For example:

$$
\bar c_p:100\rightarrow20
$$

Or:

$$
\bar c_e^{trans}:1000\rightarrow300
$$

Shocks may also enter by increasing transportation delays:

$$
\tau_e^{trans}:8h\rightarrow20h
$$

The network topology may remain unchanged while the effective physical parameters change during operation.

For the standard structure:

$$
P_i
\rightarrow
M_{ij}^{(1)}
\rightarrow
M_{ij}^{(2)}
\rightarrow
P_j
$$

If intermediate transportation capacity falls, this may lead to:

$$
q_{M_{ij}^{(1)}}(t)\uparrow
$$

And:

$$
q_{M_{ij}^{(2)}}(t)\downarrow
$$

The first inventory gradually accumulates and may reach:

$$
q_{M_{ij}^{(1)}}(t)=\bar q_{M_{ij}^{(1)}}
$$

This constrains upstream production in reverse; meanwhile, the second inventory gradually depletes and may reach:

$$
q_{M_{ij}^{(2)}}(t)=0
$$

This in turn restricts downstream production.

When an upstream $P$ supplies multiple downstream processes, allocation ratios also affect how shocks are distributed across branches. Even for the same total-output shock, different:

$$
\rho_e
$$

Can yield different depletion times in downstream inventories and different magnitudes of production decline.

Shock propagation is primarily influenced by:

$$
\boxed{
\text{Capacity}
+
\text{Inventory}
+
\text{Delay}
+
\text{Allocation}
}
$$

Together.

Specifically:

- Capacity determines the maximum local production or transportation throughput;
- Inventory determines how long the network can absorb shortfalls or accumulation;
- Delay determines the time it takes physical effects to propagate through production and transportation;
- Allocation determines how an upstream output is distributed among multiple downstream branches.

A typical propagation sequence can be written as:

$$
\boxed{
\text{local shock}
\rightarrow
\text{transport / production flow change}
\rightarrow
\text{inventory accumulation or depletion}
\rightarrow
\text{production reduction}
\rightarrow
\text{delayed downstream propagation}
}
$$

---

## 8. Items Currently Excluded from the Core Model

The following are not yet part of the core static specification:

- Supplier partnership levels;
- Credit;
- Procurement preferences;
- Bargaining power;
- Risk scores;
- Safety-stock policies;
- Reorder points;
- Target inventory;
- Work shifts and scheduling;
- Costs and prices;
- Parameters used to generate or control the network structure;
- Production and procurement decision strategies;
- Recovery and optimization strategies;
- Firm-level total-output limits shared among multiple $P$ nodes in the same company;
- Firm-level joint resource constraints such as shared equipment or labor.

These items may enter an economic, policy, control, or network-generation layer in future work without changing the basic physical specification of the core DAG.

---

## 9. Architecture Summary

This model is a production DAG centered on available material inventories $M$ and production processes $P$, with every directed edge carrying logistics behavior.

The fixed structure between successive production processes is:

$$
\boxed{
P-M-M-P
}
$$

The two $M$ nodes are distinct inventory instances, with no additional inventory nodes inserted between them.

All edges store:

$$
\boxed{
\text{Transport Capacity}
+
\text{Transport Delay}
}
$$

Input edges:

$$
\boxed{
M\rightarrow P
}
$$

Additionally store an input coefficient:

$$
\boxed{
a_{mp}
}
$$

Output edges:

$$
\boxed{
P\rightarrow M
}
$$

Additionally store an output coefficient:

$$
\boxed{
b_{pm}
}
$$

When the same output of a $P$ supplies multiple downstream processes, the corresponding $P\rightarrow M$ branches also store:

$$
\boxed{
\rho_e
}
$$

To represent the allocation ratio.

Between inventories:

$$
\boxed{
M\rightarrow M
}
$$

Only pure transportation takes place.

$P$ stores production capacity, production delay, and information about the process itself; $M$ stores a unique ID, material, state, location, maximum inventory capacity, and other information. Inventory instances with identical attributes are not automatically merged: they retain independent inventory states and dynamic histories.

The current static physical layer ultimately includes:

$$
\boxed{
\text{Inventory Identity}
+
\text{Inventory Capacity}
+
\text{Production Capacity}
+
\text{Production Delay}
+
\text{Transport Capacity}
+
\text{Transport Delay}
+
\text{Input/Output Coefficients}
+
\text{Allocation Ratios}
}
$$

As time advances, the system produces:

$$
\boxed{
\text{Inventory}
+
\text{WIP}
+
\text{In-transit Flow}
+
\text{Production Flow}
+
\text{Transport Flow}
}
$$

These dynamic states together constitute the supply chain's evolution over time and are used to study the causal propagation of local production shocks, logistics shocks, inventory buffers, multi-source inputs, and one-to-many allocation.
