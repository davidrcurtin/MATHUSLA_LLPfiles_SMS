# MATHUSLA_LLPfiles_SMS
LLP production and decay files for simulation of scalar LLP production from B decays in the SM+S model at the HL-LHC in the MATHUSLA (or other transverse LLP) detector

By David Curtin and Jaipratap Grewal, Jun 2023

BENCHMARK MODEL:

SM + singlet scalar of mass mS that mixes with 125 GeV higgs with mixing angle theta, see e.g. 
1904.10447 
for small theta, S is long-lived. S eventually decays like a long-lived SM-like higgs of mass mS, i.e. decays are yukawa-weighted to SM fermions. 

We will follow 1904.10447 reference for both computing S production and S decay rates. This is the most recent and most complete available in the literature at time of writing.

In the < 5 GeV range, a guaranteed production mechanism is exotic B meson decays via the higgs mixing, with decay rate ~ sintheta^2. This is the LLP production mode that this set of files covers. (D-meson decays give a much smaller contribution and are not included here.)

p p > B + X

B -> S + X

at 14 TeV HL-LHC

Free model parameters:
- LLP mass mS
- mixing angle between S and H, typically written as sintheta^2 = sinsqtheta. 
	Production in B/D-decays ~ sinsqtheta, S lifetimes ~ 1/sinsqtheta. 


LLP PRODUCTION:
=================
FONLL was used to calculate double-differential production cross sections ds/dpTdy for B mesons at the HL-LHC, which are used to simulate unweighted B-meson momentum vectors from the IP. 

Figure 5 from 1904.10447 shows the various exclusive branching ratios Br(B->X_i + S) where X_i are pion, kaon, and excited hadron states. These branching ratios and full 2-body decay kinematics are used to turn simulated B-meson 4-vectors into LLP 4-vectors for each different LLP mass mS. 

Each of the files SMS_LLPweight4vectorBmesonlist_mS_0.678553.csv etc in the All_SMS_LLPweight4vectors_from_B subdirectory contains a set of unweighted LLP 4-vectors for that mass mS (in GeV). Note that the phi-angles around the beam axis of these LLPs are not physically distributed, since it is assumed that LLP 4-vectors will be rotated randomly around the beam axis prior to being shot towards MATHUSLA (see instructions below). These files have format (weight, E, pX, pY, pZ) with Z = beam axis. 

The event weight is actually identical for all events in a given file, and represents (0.5) * (the number of LLPs that a single 4-vector represents for sinsqtheta = 1), so simply multiply that weight by (2 * sinsqtheta) to get the real number of LLP Production events represented by the sample. 

WARNING 1: note the required factor of 2. This is because the factor of 2 needed to obtain the number of produced B-mesons from the FONLL cross section was not included in these event weights (sorry).

WARNING 2: Note that this weight is set assuming you use all 100k 4-vectors in the file. If you use some other number N_MC, the weights should multiplied by 
(2 * sinsqtheta * 100,000/N_MC) 
so they still represent the actual number of LLPs produced at the LHC. 




LLP DECAY LENGTH:
=============
The lifetime is given in Figure 10 of 1904.10447. Below 0.2 GeV, the perturbative calculation of e.g. 1706.01920 can be used. 

For different mS in GeV, the decay length ctau in meters for sinsqtheta = 1 is supplied as a csv file in

SMS_lifetime_mSGeV_ctaumeters.csv

Simply devide the second column by sinsqtheta to get the decay length for your mixing angle. 



LLP DECAY:
=========== 

This is very tricky for this scalar LLP for mS ~ GeV due to large hadronization uncertainties. We will closely follow 1904.10447 with some modifications for practical monte carlo simulation. In particular, we will use some branching ratios from Figure 10 (left). The needed ones are supplied in csv files in the S_br_tables subdirectory (mumu, tautau, pipi, ss, cc, gg).

Unweighted decay events are supplied in the LLP restframe in the SMS_LLP_decays_geant subdirectory. 
Files like cc_4.7.txt
contain LLP decays for the LLP mass indicated after the "_", with different LLP decays separated by two blank lines
the first row of each event block is the original LLP 4-vector used in the simulation. This can be ignored (since the simulation used for LLP decays does not match the kinematic distribution of actual LLP production at the HL-LHC.)
Subsequent rows contain lists of LLP decay final states IN THE LLP RESTFRAME, in format 
E, pX, pY, pZ, mass, PID, particle name. note not all PIDs have a 'particle name' entry, that column is just for convenience. 
Information on how these decays were generated is below in A.3. 


Decay instructions for each LLP mass window:

- for 2m_e < m_S < 2m_mu, assume 100% decay to ee
- for 2m_mu < m_S < 2 m_pi, assume 100% decay to mumu
- for 2m_pi < mS < 0.7 GeV, follow br curves for relative fractions of mumu and pipi decays. 
- for 0.7 GeV < mS < 2mK = 0.98 GeV, use Br(S->mumu) for fraction of decays to mumu. Assume Br(S->hadrons) = 1 - Br(S-mumu), and for that fraction of decays, use the decay files hadrons0.7to0.98gev_XX.txt
- for 0.98 GeV = 2mK < mS < 2 GeV, same as previous, but use hadrons1to2gev_XX.txt files. 
- for 2 GeV < mS < 5 GeV, follow the Br curves for relative fractions of S -> gg, cc, tautau, ss, mumu. For each mode, use corresponding decay files gg_XX.txt, cc_XX.txt etc.



Note: The precise S branching fractions in the 0.7 -  2 GeV mS window are subject to significant uncertainties, but the actual effect on reconstruction efficiencies should not be major, you can estimate it by comparing efficiency at 0.7 vs 2 GeV. Since reconstruction efficiency should increase roughly monotonically with mass, the difference between the start and end of the hadronic uncertainty window should give an idea of the relative uncertainty within the window. 


Note: these decay files INCLUDE the invisible decays, e.g. decays to only active neutrinos or only pi0s. Therefore, if you feed these files directly to the LLP gun, you will NOT have to include any Br(vis) renormalization factor in the final number of observed LLP events. For computational efficiency, you might make versions of these files that only include the decays with charged final states: in that case, adjust all the weights down by Br(vis), the fraction of decays with charged final states.


USAGE FOR MATHUSLA GEANT SIMULATIONS
=======================================

Select some LLP mass mS to study, say mX = 3.06185 GeV. 
You will need the corresponding LLP 4-vector file
SMS_LLPweight4vectorBmesonlist_mS_3.06185.csv
and the decay products files for the relevant decay modes, see above. 


1. Define rapidity and azimuthal angle ranges ((eta_min, eta_max), (phi_min, phi_max)) that roughly bracket the solid angle acceptance of MATHUSLA. (Overestimate so that no part of MATHUSLA falls outside of that range.)

2. Select a RANDOM (preferably not sequential, but also avoid duplicates) subset of LLP 4-vectors called {MCsample} from SMS_LLPweight4vectorBmesonlist_mS_3.06185.csv that you want to use in your simulations, or use all of them. Let the number of 4-vectors in this sample be N_MC. 

Each LLP 4-vector i in this sample {MCsample} has weight
w_i = (2 * sinsqtheta * 100,000/N_MC) 
This weight corresponds to the number of LLPs that this simulated 4-vector represents. Note you do not need to specify sinsqtheta right now, since it won't affect kinematics. Later, it will set overall production rate and LLP lifetime. 


3. From {MCsample}, select the subset {MCsample_etacut} that has rapidity in the range (eta_min, eta_max).

Furthermore, due to the symmetry around the beam axis, the phi-angle can be freely chosen for each event. For each surviving LLP 4-vector in {MCsample_etacut}, choose a random phi-angle in the range (phi_min, phi_max) and rotate the 4-vector around the beam axis to have that angle. The chance that a random LLP has a phi angle in the MATHUSLA acceptance is 
f_phi = (phi_max - phi_min)/2pi

Give each rotated LLP 4-vector i in our sample {MCsample_etacut} a weight
w'_i = w_i * f_phi. 
This weight corresponds to the number of LLPs flying through (or close to) MATHUSLA that this simulated 4-vector represents. 


4. Now we have to choose sinthetasq. This will numerically set all the weights wi and set the lifetime in accordance with the lifetime curve in SMS_lifetime_mSGeV_ctaumeters.csv.

For each LLP 4-vector i in {MCsample_etacut}, find lengths along its trajectory (L1, L2) where it enters and exits the MATHUSLA detector volume that we include in our analysis. The chance of the LLP to decay in the detector is then

P_decay_i  = Exp(-L1/(b*ctau)) - Exp(-L2/(b*ctau))

where b = sqrt(px^2+py^2+pz^2)/mX is the boost of the LLP. 

Next, choose an  LLP decay position along the LLP trajectory between L1 and L2 from an exponential distribution
dPdecay/dl = 1/(b*ctau) Exp[-l/(b*ctau)]. 

Finally, choose a random number between 0 and 1, and according to the outcome, choose an LLP decay mode according to the "LLP Decay" instructions above, and for that LLP decay mode, pick a decay event at random from the corresponding decay file. 

Lorentz-transform the decay daughter products into the lab-frame along the LLP momentum direction with boost b. Place them at decay position (x,y,z), and assign this decay event the weight 
w''_i = w'_i * P_decay_i
which is just the number of LLP decays this decay event represents. 

5. Do whatever you do in your GEANT analysis to decide whether MATHUSLA can reconstruct this LLP decay i. If yes, count this LLP decay as w''_i observed LLP decays. 


6. 
FOR RECONSTRUCTION EFFICIENCY STUDIES: 
do the above, separate LLP decay events into groups (reconstructed) and (not reconstructed). 
efficiency = sum_i(reconstructed) w_i / sum_i(reconstructed + not reconstructed) w_i

This efficiency will have a significant dependence on mS, and will in general depend on LLP decay length ctau. However, in the long lifetime limit (say ctauX > ~ 1km, or sinsqth < 10^-15 for mS < 5 GeV), LLP decays are approximately uniformly distributed along their trajectories in MATHUSLA, and the lifetime will drop out of the efficiency. This efficiency in the 'long lifetime limit' is therefore a particularly interesting quantity to compute in simulations. 


FOR SIGNAL vs BACKGROUND and SENSITIVITY STUDIES:
For each mS, and for each sinsqtheta, obtain the list of observed LLP decays in our simulation, and check whether
sum_i w''_i >  4 (or whatever the minimum required number of observed LLP decays for our analysis is)






====================


A.1 DETAILS ON B, D MESON PRODUCTION
====================================

use public FONLL website to compute ds/dpt^2dy (pb/GeV^2)

# Job started on: Tue Mar  7 16:49:29 CET 2023 .
# FONLL heavy quark hadroproduction cross section, calculated on Tue Mar  7 16:49:36 CET 2023
# FONLL version and perturbative order: ## FONLL v1.3.2 fonll [ds/dpt^2dy (pb/GeV^2)]
# quark = bottom
# final state = meson. NP params (cm,lm,hm) = 24.2, 26.7, 22.2
# BR(q->meson) = 1
# ebeam1 = 7000, ebeam2 = 7000
# PDF set = CTEQ6.6



# Job started on: Tue Mar  7 16:59:27 CET 2023 .
# FONLL heavy quark hadroproduction cross section, calculated on Tue Mar  7 16:59:34 CET 2023
# FONLL version and perturbative order: ## FONLL v1.3.2 fonll [ds/dpt^2dy (pb/GeV^2)]
# quark = charm
# final state = meson (meson = 0.7 D0 + 0.3 D+). NP params (cm,lm,hm) = 0.1, 0.06, 0.135
# BR(q->meson) = 1
# ebeam1 = 7000, ebeam2 = 7000
# PDF set = CTEQ6.6

Given that LHC events are rotationally symmetric with respect to rotations around the beam axis, the binned dsigma/dpTdy differential cross section can therefore be used to construct a CDF and hence randomly generate B and D-meson momenta, which is what we did to get B- and D-meson 3-momentum-vectors. For D mesons, we assume 70% are D0 and 30% D+. For B-mesons, we assume 44.8% B0, 44.8% B+/-, 10.3% Bs0 and 0.018% Bc+/-. The B-meson fractions are taken from pythia8 hadronization, and are compatible with LHC measurements /910.09934. The resulting B- and D-meson 4-vectors were then decayed to various LLPs using full 2- and 3-body kinematics (phase space only). 


Note that FONLL computes cross sections to mean "average of q and qbar + X production", so to get actual number of B/D-mesons produced at LHC (1710.04921), we have to multiply the FONLL cross sections by 2. This is NOT included in the weighted LLP event generation, and has to be added (see description above). 


A.2 EXOTIC HIGGS DECAYS
======================

The SM+S model generically includes the quartic coupling lambda_HS H^2 S^2, which leads to an exotic higgs decay h->SS with coupling ~ lambda_HS v. This can give very significant rates of S production, especially above 5 GeV where it would dominate. We do not include this lambda_HS-dependent LLP production mode in these files. For a given lambda_SH, the HXX model can be used to obtain MATHUSLA events for LLP production from exotic Higgs decay.



A.3 DETAILS ON DECAY EVENT GENERATION
=======================================

Decays were simulated in a modified SM madgraph model that includes the hgg effective vertex for scalar-gluon-gluon couplings, treating the h as the LLP. They were hadronized in pythia8, and then the decay products of the h were extracted and transformed to the h restframe. The madgraph model and param_cards can be found in madgraph_cards_for_S_decay subdirectory. 


// for h->hadron decays in uncertain 0.7 - 0.98 GeV gev mass window
./bin/mg5_aMC
import model sm_allyuk_hgg
generate mu- mu+ >  h a, h > gg
output proc_SMS_decay_hadrons0.7to0.98gev
quit

// for h->hadron decays in uncertain 1 - 2 GeV gev mass window
./bin/mg5_aMC
import model sm_allyuk_hgg
define finalstate = g s~ s
generate mu- mu+ >  h a, h > finalstate  finalstate
output proc_SMS_decay_hadrons1to2gev
quit


// for h->gg decays above 2 gev
./bin/mg5_aMC
import model sm_allyuk_hgg
generate mu- mu+ >  h a, h > g g
output proc_SMS_decay_gg
quit

// for h->ss decays above 2 gev
./bin/mg5_aMC
import model sm_allyuk_hgg
generate mu- mu+ >  h a, h > s s~
output proc_SMS_decay_ss
quit


// for h->cc decays above 3.8 gev 
./bin/mg5_aMC
import model sm_allyuk_hgg
generate mu- mu+ >  h a, h > c c~
output proc_SMS_decay_cc
quit


// for h->tau tau decays above 3.6 gev
./bin/mg5_aMC
import model sm_allyuk_hgg
generate mu- mu+ >  h a, h > ta+ ta-
output proc_SMS_decay_tautau
quit
