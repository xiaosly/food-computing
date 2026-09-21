# The model at a glance

This note states what is modelled, with the governing equations. It is a summary of the Methods of
the manuscript in preparation. Parameter values are the central case; every experiment in the
working repository records its own parameters beside its outputs.

## 1. Heat sources

US data-center campuses and announced projects from a public catalogue are aggregated into **393
source cells** on a 25 km equal-area grid (contiguous 48 states). The cells are regional proxies, not
metered facilities. National IT capacity added after 2026 follows declared scenarios; the central
increments are 114.9 GW in 2035 and 173.6 GW in 2050. With IT utilisation 0.65 and a recoverable
fraction of 0.25, each added GW of IT gives 5.12 PJ of recoverable heat per year, at a source
temperature of 53 °C (a liquid-cooling design assumption).

Hourly energy available at cell $k$ for a national increment $D$:

$$C_{k,t}(D)=\frac{P_k\,\Delta t\,D}{D_{\mathrm{ref}}}$$

where $P_k$ is the cell's reference thermal power and the proportions among cells stay fixed.

## 2. Food heat demand

Two users with different temperatures, schedules and equipment compete for the same source heat.

$$Q^{\mathrm{CEA}}_{j,t}=A_j\,h_{j,t}\qquad\qquad Q^{\mathrm{proc}}_{j,t}=Q_{j,\mathrm{yr}}\,\phi_{j,t}$$

- **Controlled-environment agriculture.** $A_j$ is protected-crop area from USDA statistics, split
  into ten crop categories by state. $h_{j,t}$ is an hourly heat intensity from a reduced greenhouse
  envelope driven by NASA POWER hourly weather (2021–2023) and crop-specific temperature and light
  recipes. The envelope was fitted to one greenhouse's heat-meter record and is cross-checked
  against the mechanistic GreenLight model over the same weather and setpoints.
- **Food processing.** $Q_{j,\mathrm{yr}}$ is useful heat for NAICS 311 boiler and process heating
  at up to 70 °C, from NREL county-level manufacturing data (2,233 counties, 15,865 temperature and
  end-use segments), converted from fuel with end-use efficiencies. $\phi_{j,t}$ is a normalized
  hourly operating profile.

## 3. Connection and heat pumps

A demand connects to its nearest same-state source cell within 25 km; unconnected demand stays in
the denominator with full backup. Pipe efficiency and heat-pump performance are

$$\eta_{\mathrm{pipe}}=\max(0.75,\;1-0.0025\,d)\qquad\qquad
\mathrm{COP}_j=\min\left[6,\;\frac{0.5\,(T_{j,\mathrm{hot}}+273.15)}{T_{j,\mathrm{hot}}-53}\right]$$

with $d$ in km. Below 53 °C supply is direct. Above it, each unit of useful heat draws
$S_j=(1-1/\mathrm{COP}_j)/\eta_{\mathrm{pipe}}$ of source heat and $W_j=1/\mathrm{COP}_j$ of
electricity; supply is unavailable below a COP of 2.

## 4. Hourly dispatch

Within each cell and hour every connected user receives the same fraction of its request, so neither
user has priority or a second copy of the source budget:

$$f_{k,t}=\min\left[1,\;\frac{C_{k,t}}{\sum_{j\in k}S_j\,Q_{j,t}}\right]\qquad\qquad q_{j,t}=f_{k,t}\,Q_{j,t}$$

Backup supplies $Q_{j,t}-q_{j,t}$. The account closes in every cell and hour:

$$Q_{\mathrm{source}}+E_{\mathrm{HP}}=Q_{\mathrm{useful}}+Q_{\mathrm{loss}}$$

Pump electricity (0.0072 MWh per MWh useful) is carried separately. The base dispatch has no storage
and no transfer between cells.

## 5. Siting

New cultivation and new processing capacity is placed by optimization, re-solved for every year from
2026 to 2050 with the source geography held fixed.

- **Cultivation.** Candidate land is screened with NLCD land cover, PAD-US protected areas, slope
  from elevation data and a connected-parcel rule. The problem is solved lexicographically: first
  the monthly food-heat service that can be completed, then the heating requirement of the completed
  task, which depends on the climate of wherever the crop is placed.
- **Processing.** New tasks are tied to industry-matched anchors from the EPA facility registry,
  with regional activity limits, finite plant duty and conditional water and wastewater utilities.
  Source heat and auxiliary electricity already reserved for cultivation are debited first.
- **Local construction.** A separate local model follows persistent construction decisions over
  time for compatible campus pairs, with cold inventory, handling and delivery.

Optimizations are solved with Gurobi and independently re-solved with HiGHS or checked against
analytic optima.

## 6. Storage and the cold chain

Intraday thermal buffering is replayed over each year's layout with finite capacity (0.3 to 0.7 kWh
per m², six-hour charge and discharge limits, 24-hour lookahead). The additional useful heat equals
the backup avoided.

The cold chain is a **local, electric** account: precooling, 4 °C cold rooms and refrigerated
delivery per tonne delivered, with sensitivities on refrigeration COP, door air exchange and
residence time. Data-center heat serves cultivation; refrigeration remains electric, and this
account is not scaled to the nation.

## 7. Economics

The matched conventional supply is natural gas, with separate efficiencies for greenhouses (0.85),
processing boilers (0.75) and direct process heating (0.50). With avoided fuel expenditure $G$,
heat-pump and pump electricity payments $E$ and a source-heat cost $O$, the capital the heat
interface can bear per peak useful kW is

$$K_{\max}=\frac{G-E-O}{\mathrm{CRF}\;P_{\mathrm{peak}}}\qquad\qquad
\mathrm{CRF}=\frac{r}{1-(1+r)^{-N}}$$

with $r=0.08$ and $N=25$ years in the central case. A source passes when its own ceiling reaches the
benchmark capital cost. A common price surface varies gas (10 to 50 USD per MWh) and electricity (40
to 240 USD per MWh) with the dispatch held fixed. Transfers among producers, interface investors and
electricity providers divide this partial account without creating it. Filed utility tariffs are used
as separate billing checks, not scaled up.

## 8. Interruption experiments and outage records

Native GreenLight tomato simulations in four climates (Minnesota, Ohio, New Mexico, Florida) pair an
uninterrupted control with (i) loss of heat and lighting and (ii) loss of lighting while heat stays
available, for 24, 72 and 168 hours starting in each climate's maximum-heating window. Harvest is
the integral of the model's fruit harvest flux. Sub-freezing hours lie outside the crop model's
validated range, so those harvest contrasts are qualitative.

DOE EAGLE-I county-level outage records (15-minute resolution, 2014–2025) are used to place the
tested durations in the observed distribution of county outage streaks. They describe county
events, not the interruption probability of any facility, and the project has no incidence model.

## 9. What is outside the model

Cooling and dehumidification loads of greenhouses, whole-facility costs, food prices and demand
response, observed contracts, facility-level reliability, and any uncertainty quantification beyond
deterministic scenario sweeps.
