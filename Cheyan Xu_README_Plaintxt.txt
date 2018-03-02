Overall assumptions:
1. Palsson’s model is correct
2. Steady state has been reached for all intracellular species

A. No regulations
	1.	With O2 
	a. Function “maximize_cellmass_data_dictionary” was used in the solve.jl file
	b. The glucose uptake rate was first set to be 2.4 mmol/(gdw-hr) because this was the value available online (Also included in the excel sheet for problem 1)
	However, 2.4 only works for the aerobic condition. Under anaerobic condition, it would not generate an optimal solution. Thus, I increased this value
	from 2.4 to 20 which works pretty well in both cases.
	c. According to Palsson’s paper (p24), the ATP maintenance reaction was represented with a lower bound of 8.39 mmol/gdw-hr
	d. The upper bounds of the exchange array were randomly given a positive number (100 in this case)

Codes under the function “maximize_cellmass_data_dictionary”:

function maximize_cellmass_data_dictionary(time_start,time_stop,time_step)

	# Get the default data_dictionary -
	data_dictionary = DataDictionary(time_start,time_stop,time_step)

	# setup the obj -
	objective_coefficient_array = data_dictionary["objective_coefficient_array"]
	objective_coefficient_array[24] = 1.0

	# Default flux bounds array -
	default_flux_bounds_array = data_dictionary["default_flux_bounds_array"]
	default_flux_bounds_array[21,2] = 0.0


	# ATP maintenance -
	default_flux_bounds_array[20,1:2] = 8.39


	# setup exchange array -

	species_bounds_array = data_dictionary["species_bounds_array"]
	exchange_array = [
		0.0	100.0	;	# 73 M_ac_b
		0.0	100.0	;	# 74 M_acald_b
		0.0	100.0	;	# 75 M_akg_b
		0.0	100.0	;	# 76 M_co2_b
		0.0	100.0	;	# 77 M_etoh_b
		0.0	100.0	;	# 78 M_for_b
		0.0	100.0	;	# 79 M_fru_b
		0.0	100.0	;	# 80 M_fum_b
		-20	100.0	;	# 81 M_glc_D_b
	
		#The glucose uptake rate was set to be 2.4 at first then changed to -20 in order to get an optimal solution for the LP

		0.0	100.0	;	# 82 M_gln_L_b
		0.0	100.0	;	# 83 M_glu_L_b
		-10.0	100.0	;	# 84 M_h2o_b
		-100.0	100.0	;	# 85 M_h_b
		0.0	100.0	;	# 86 M_lac_D_b
		0.0	100.0	;	# 87 M_mal_L_b
		-100.0	100.0	;	# 88 M_nh4_b

		# 0 for anaerobic and -20 for aerobic

		-20.0	100.0	;	# 89 M_o2_b
		-100.0	100.0	;	# 90 M_pi_b
		0.0	100.0	;	# 91 M_pyr_b
		0.0	100.0	;	# 92 M_succ_b
	]


After running the “solve.jl” file, the growth rate under 20 mmol/gDW-hr was calculated to be 1.209 hr-1

######################################################################################################################################

		2. Without O2
		a. The oxygen uptake rate was set to 0 for the obvious reason

Codes under the function “maximize_cellmass_data_dictionary”:

function maximize_cellmass_data_dictionary(time_start,time_stop,time_step)

	# Get the default data_dictionary -
	data_dictionary = DataDictionary(time_start,time_stop,time_step)

	# setup the obj -
	objective_coefficient_array = data_dictionary["objective_coefficient_array"]
	objective_coefficient_array[24] = 1.0

	# Default flux bounds array -
	default_flux_bounds_array = data_dictionary["default_flux_bounds_array"]
	default_flux_bounds_array[21,2] = 0.0


	# ATP maintenance -
	default_flux_bounds_array[20,1:2] = 8.39


	# setup exchange array -

	species_bounds_array = data_dictionary["species_bounds_array"]
	exchange_array = [
		0.0	100.0	;	# 73 M_ac_b
		0.0	100.0	;	# 74 M_acald_b
		0.0	100.0	;	# 75 M_akg_b
		0.0	100.0	;	# 76 M_co2_b
		0.0	100.0	;	# 77 M_etoh_b
		0.0	100.0	;	# 78 M_for_b
		0.0	100.0	;	# 79 M_fru_b
		0.0	100.0	;	# 80 M_fum_b
		-20	100.0	;	# 81 M_glc_D_b
	
		#The glucose uptake rate was set to be 2.4 at first then changed to -20 in order to get an optimal solution for the LP

		0.0	100.0	;	# 82 M_gln_L_b
		0.0	100.0	;	# 83 M_glu_L_b
		-10.0	100.0	;	# 84 M_h2o_b
		-100.0	100.0	;	# 85 M_h_b
		0.0	100.0	;	# 86 M_lac_D_b
		0.0	100.0	;	# 87 M_mal_L_b
		-100.0	100.0	;	# 88 M_nh4_b

		# 0 for anaerobic and -20 for aerobic

		0	100.0	;	# 89 M_o2_b
		-100.0	100.0	;	# 90 M_pi_b
		0.0	100.0	;	# 91 M_pyr_b
		0.0	100.0	;	# 92 M_succ_b
	]

After running the “solve.jl” file, the growth rate under 20 mmol/gDW-hr was calculated to be 0.515 hr-1

Difference between aerobic and anaerobic conditions after calculations:
During the TCA cycle, O2 is needed in the electron transport chain that is upstream of the cycle to regenerate NAD+ from NADH which acts as a co-enzyme. 
Under anaerobic conditions, the TCA cycle functions not as a cycle but as two separate pathways. However, under the aerobic condition, this acts as a cycle. 
Additionally, anaerobic process tends to generate more side products comparing to the aerobic process. 

##############################################################################################################################################


B. Regulations
	
	Under boolean regulation condition, according to Palsson, “If the genes associated with an enzyme or transport protein/complex are repressed, then in silico flux is constrained to zero for the corresponding reaction”, “A gene is considered to be induced when evaluation of the corresponding Boolean rule gives “trues”. Then, by using table 16 and 17, I would be able to analyze if the genes was repressed or not. As a result, ICL, FUMt2_2, MALt2_2 and SUCCr2_2, FORT2, FORTi, FRD7, and PFL 
would be turned offed after comparing these two table carefully for the aerobic condtion; ICL, MALS, FUMt2_2, MALt2_2 and SUCCr2_2,D-LACt2, MDH, and NADH16 would be turned off under the anaerobic condition.


	1.	With O2
ICL, FUMt2_2, MALt2_2 and SUCCr2_2, FORt2, FORti, FRD7, and PFL were turned off under the aerobic condition
	
Codes under the function “maximize_cellmass_data_dictionary”:

function maximize_cellmass_data_dictionary(time_start,time_stop,time_step)

	# Get the default data_dictionary -
	data_dictionary = DataDictionary(time_start,time_stop,time_step)

	# setup the obj -
	objective_coefficient_array = data_dictionary["objective_coefficient_array"]
	objective_coefficient_array[24] = 1.0

	# Default flux bounds array -
	default_flux_bounds_array = data_dictionary["default_flux_bounds_array"]
	default_flux_bounds_array[21,2] = 0.0


	# ATP maintenance -
	default_flux_bounds_array[20,1:2] = 8.39


	# setup exchange array -

	species_bounds_array = data_dictionary["species_bounds_array"]
	exchange_array = [
		0.0	100.0	;	# 73 M_ac_b
		0.0	100.0	;	# 74 M_acald_b
		0.0	100.0	;	# 75 M_akg_b
		0.0	100.0	;	# 76 M_co2_b
		0.0	100.0	;	# 77 M_etoh_b
		0.0	100.0	;	# 78 M_for_b
		0.0	100.0	;	# 79 M_fru_b
		0.0	100.0	;	# 80 M_fum_b
		-20	100.0	;	# 81 M_glc_D_b	

		#The glucose uptake rate was set to be 2.4 at first then changed to -20

		0.0	100.0	;	# 82 M_gln_L_b
		0.0	100.0	;	# 83 M_glu_L_b
		-10.0	100.0	;	# 84 M_h2o_b
		-100.0	100.0	;	# 85 M_h_b
		0.0	100.0	;	# 86 M_lac_D_b
		0.0	100.0	;	# 87 M_mal_L_b
		-100.0	100.0	;	# 88 M_nh4_b

		# 0 for anaerobic and -20 for aerobic

		-20	100.0	;	# 89 M_o2_b
		-100.0	100.0	;	# 90 M_pi_b
		0.0	100.0	;	# 91 M_pyr_b
		0.0	100.0	;	# 92 M_succ_b
	]

	# Reactions that are repressed under aerobic condition were set to 0
	default_flux_bounds_array[65,2] = 0
	default_flux_bounds_array[66,2] = 0
	default_flux_bounds_array[67,2] = 0	
	default_flux_bounds_array[71,2] = 0
	default_flux_bounds_array[90,2] = 0
	default_flux_bounds_array[94,2] = 0
	default_flux_bounds_array[129,2] = 0
	default_flux_bounds_array[107,2] = 0

After running the “solve.jl” file, the growth rate under 20 mmol/gDW-hr was calculated to be 1.209 hr-1

##############################################################################################################################################################################

	2.	Without O2
ICL, MALS, FUMt2_2, MALt2_2 and SUCCr2_2,D-LACt2, MDH, NADH16 were turned off under the anaerobic condition

function maximize_cellmass_data_dictionary(time_start,time_stop,time_step)

	# Get the default data_dictionary -
	data_dictionary = DataDictionary(time_start,time_stop,time_step)

	# setup the obj -
	objective_coefficient_array = data_dictionary["objective_coefficient_array"]
	objective_coefficient_array[24] = 1.0

	# Default flux bounds array -
	default_flux_bounds_array = data_dictionary["default_flux_bounds_array"]
	default_flux_bounds_array[21,2] = 0.0


	# ATP maintenance -
	default_flux_bounds_array[20,1:2] = 8.39


	# setup exchange array -

	species_bounds_array = data_dictionary["species_bounds_array"]
	exchange_array = [
		0.0	100.0	;	# 73 M_ac_b
		0.0	100.0	;	# 74 M_acald_b
		0.0	100.0	;	# 75 M_akg_b
		0.0	100.0	;	# 76 M_co2_b
		0.0	100.0	;	# 77 M_etoh_b
		0.0	100.0	;	# 78 M_for_b
		0.0	100.0	;	# 79 M_fru_b
		0.0	100.0	;	# 80 M_fum_b
		-20	100.0	;	# 81 M_glc_D_b	

		#The glucose uptake rate was set to be 2.4 at first then changed to -20

		0.0	100.0	;	# 82 M_gln_L_b
		0.0	100.0	;	# 83 M_glu_L_b
		-10.0	100.0	;	# 84 M_h2o_b
		-100.0	100.0	;	# 85 M_h_b
		0.0	100.0	;	# 86 M_lac_D_b
		0.0	100.0	;	# 87 M_mal_L_b
		-100.0	100.0	;	# 88 M_nh4_b

		# 0 for anaerobic and -20 for aerobic

		0	100.0	;	# 89 M_o2_b
		-100.0	100.0	;	# 90 M_pi_b
		0.0	100.0	;	# 91 M_pyr_b
		0.0	100.0	;	# 92 M_succ_b
	]

	#Reactions that are repressed under aerobic condition were set to 0
	#default_flux_bounds_array[65,2] = 0
	#default_flux_bounds_array[66,2] = 0
	#default_flux_bounds_array[67,2] = 0	
	#default_flux_bounds_array[71,2] = 0
	#default_flux_bounds_array[90,2] = 0
	#default_flux_bounds_array[94,2] = 0
	#default_flux_bounds_array[129,2] = 0
	#default_flux_bounds_array[107,2] = 0

	#Reactions that are repressed under anaerobic condition were set to 0
	default_flux_bounds_array[29,2] = 0
	default_flux_bounds_array[30,2] = 0
	default_flux_bounds_array[71,2] = 0
	default_flux_bounds_array[90,2] = 0
	default_flux_bounds_array[93,2] = 0
	default_flux_bounds_array[94,2] = 0
	default_flux_bounds_array[95,2] = 0
	default_flux_bounds_array[96,2] = 0
	default_flux_bounds_array[99,2] = 0
	default_flux_bounds_array[129,2] = 0

After running the “solve.jl” file, the growth rate under 20 mmol/gDW-hr was calculated to be 0.515 hr-1

############################################################################################################################################################################

Difference between aerobic and anaerobic conditions under regulations after calculations:
As we can see, after adding the regulations to the system. the growth rate calculated didn’t change. This indicates that these reactions wouldn’t happen anyways in the process under each condition.
