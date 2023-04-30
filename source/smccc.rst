SMC Calling Convention (SMCCC)
----------------------------------

HypX can't do everything on it's own. Instead, it has to make calls to the EL3 monitor in the trustzone do things for it (i.e. PSCI calls, random number generation, etc.)

The purpose of this document is to describe the SMC calling convention and how SMCs are constructed. 