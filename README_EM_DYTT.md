# EMEP MSC-W ACTM rv5.0_EM: adaptation of rv5.0 to include HCl chemistry,assuming externally mixed sea salt

Changes made by Damaris Y. T. Tan (damtan@ceh.ac.uk / damaris.tan@ed.ac.uk)
UK Centre for Ecology & Hydrology / University of Edinburgh 
Published on Zenodo November 2026.
Paper describing model changes to be submitted to ACP shortly.

Changes in the code are marked by the author's initials (DYTT). 
The initials may be hyphenated with an additional letter combination,
e.g. ''DYTT-SSfreac''.
Please ignore the term after the hyphen - this was part of the model 
adaptation process and is no longer relevant.

Original model code (Simpson et al., 2012) is available from the Norwegian
Meteorological Institute GitHub page: https://github.com/metno/emep-ctm .

## Overview

HCl chemistry has been added, assuming external mixing of fine sea salt. 

HCl is produced via direct emissions, and via chemical reaction:
HNO3 + fine sea salt -> HCl + fine NaNO3
HNO3 + coarse sea salt -> HCl + coarse NaNO3 (this is a pre-existing reaction, 
but HCl production was not accounted for.)

HCl equilibrium with NH3 is also accounted for via ISORROPIA or EQSAM 
equilibrium modules. 

HCl is removed through wet and dry deposition. 

## Definitions of new species

3 new species were added:

HCl gas.
pCl: defined as the fine chloride associated with NH4Cl.
pNO3_f: defined as the fine NO3 associated with fine NaNO3. 

## Summary of changes in individual modules

### AeroConstants_mod.f90

This code only works with the ISORROPIA and EQSAM equilibrium modules. 
It does not work with MARS.
EQSAM has better agreement with measurements, though this strongly 
depends on the choice of uptake coefficient (GAM) for the reaction 
between sea salt and HNO3 to form HCl (defined in CM_ChemRates_mod.f90). 
For more details, see the paper to be submitted shortly. 
ISORROPIA may work better if paired with a different uptake coefficient. 

The INTERNALMIXED flag is set to .false., because this is the assumption 
made by this model version. This flag is not, however, used in the 
code. Setting this to .true. will change nothing.

### AerosolCalls.f90

By definition, this code works on the assumption of external mixing, 
i.e. AERO%INTERNALMIXED = F. Hence all possibilities for sea salt 
internal mixing options are commented out. 

### Aqueous_n_WetDep_mod.f90

Added pNO3_f to all processes involving NO3_f.

### ChemFunctions_mod.f90

Added pNO3_f to all processes involving NO3_f.

### CM_ChemDims_mod.f90

Changed array dimensions to account for the addition of new species, emissions, 
and reaction rates. 

### CM_ChemGroups_mod.f90

Added pCl and pNO3_f to various groups. 
Note that I have NOT included pNO3_f, because it is associated with NaNO3. 
The Na comes from sea salt, which is not included here. However, I think maybe 
pNO3_f should actually be included...?

### CM_ChemRates_mod.f90

Changed the uptake coefficient of HNO3 onto a coarse sea salt particle 
(GAM) from 0.01 to 0.03. 
Added a reaction rate of for the reaction between HNO3 and fine sea 
salt (analagous to the reaction rate between HNO3 and coarse sea salt, 
but using the surface area of fine sea salt.)

### CM_ChemSpecs_mod.f90

Added HCl, pCl and pNO3_f species.

### CM_DryDep.inc

HCl deposits like HCl (i.e. same properties as HNO3, but with diffusivity 
in air specific to HCl).
pCl (which is associated with NH4Cl) is assumed to deposit like NH4NO3 (i.e. 
like all other fine PM, but with the near-surface evapouration effect.)
pNO3_f deposits like fine sea salt, because this is NO3_f associated with 
NaNO3. This is formed on fine sea salt (NaCl (fine) + HNO3 -> HCl + NaNO3 (fine)), 
which is external mixed and therefore deposits like all other SSf. 

### CM_EmisFile.inc

Added the possibility for HCl and pCl files.

### CM_EmisSpecs.inc

Added the possibility for HCl and pCl emissions.

### CM_Reactions1.inc

Added loss of HNO3 via reaction with fine sea salt. (Loss via coarse sea salt 
already existed.)
Added HCl production via direct emissions, and reaction between HNO3 and sea salt
(both fine and coarse).
Added pCl production via direct emissions.

### CM_Reactions2.inc

Added pNO3_f production via HNO3 + fine sea salt reaction. 
NO3_C production via HNO3 + coarse sea salt already exists.

### CM_WetDep.inc

HCl wet dep is like HNO3, following approach by Wang et al. (2025).
pCl wet dep is like all other (non-sea salt) fine PM.
pNO3_f deposits like fine seasalt. This is because this is the NO3_f associated 
with NaNO3 (fine), formed via HNO3 + NaCl (fine) --> HCl + NaNO3 (fine) reaction.
Since sea salt is externally mixed, NaNO3 will be part of the fine sea salt particle, 
and hence deposits like fine sea salt.

### Config_module.f90

*** Changed the end of the day to 00:00 to correspond with UK measurements.
This may need to be changed back to 06:00 to correspond with EMEP measurements.***

Added in HCl, pCl and pNO3_f where required. 

### DryDep_mod.f90

Amended this to ensure that both NH4NO3 and NH4Cl deposit in the same way, 
i.e. accounting for the near-surface evaporation effect. 
Required making sure the correct fraction of NH4 evaporates in this way.

### EQSAM_mod.f90

Added HCl as an input. EQSAM already had pre-existing capability for dealing
with HCl. 
Had to make sure that order of subroutine arguments corresponds to order in 
AerosolCalls.f90!!!

### GasParticleCoeffs_mod.f90

Added gas properties for HCl - same as for HNO3 (Wang et al., 2025)
but using a diffusivity specific to HCl of 0.16 cm2/s at STP, which is 
the literature value quoted in Meyer (2023).

### My_Derived_mod.f90

Added pNO3_f where NO3 is accounted for. 

### Setup_1d_mod.f90

Added pNO3_f analogous to NO3_F.


## References

Simpson, D., Benedictow, A., Berge, H., Bergström, R., Emberson, L. D., Fagerli, H., Flechard, C. R., Hayman, G. D., Gauss, M., Jonson,J. E., Jenkin, M. E., Nyíri, A., Richter, C., Semeena, V. S., Tsyro, S., Tuovinen, J.-P., Valdebenito, A., and Wind, P.: The EMEP MSC-W chemical transport model - technical description, Atmospheric Chemistry and Physics, 12, 7825–7865, https://doi.org/10.5194/acp-12-7825-2012, 2012.

Wang, Y., Nemitz, E., Tomlinson, S. J., Carnell, E. J., Yao, L., Scheffler, J., Liska, T., Pearson, C., Dragosits, U., Venkataraman, C., Balasubramanian, S., Beck, R., Sutton, M. A., and Vieno, M.: Response of South Asia PM2.5 pollution to ammonia emission changes and associated impacts on human health, Environment International, 195, 109207, https://doi.org/10.1016/j.envint.2024.109207, 2025.

Meyer, P.: Quantitative Determination of the Diffusion Constant of Hydrogen Chloride in Air: A Way to Eliminate a (School-Based) Misconception and to Re-evaluate the Counterdiffusion of HCl and NH3, Journal of Chemical Education, 100, 4941–4945, https://doi.org/10.1021/acs.jchemed.3c00778, 2023.