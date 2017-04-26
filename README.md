###############################################################
#
# Autoignition of a methane air mixture at stoichiometry
# and atmospheric pressure, for different
# initial temperature
#
###############################################################
	
	import sys
	import numpy as np
	from cantera import *
	import cantera as ct
#################################################################
#Mechanism used for the process
	
	gas = Solution('gri30.cti')
#Initial temperature, Pressure and stoichiometry
	
	gas.TPX = 1250, one_atm, 'CH4:0.5,O2:1,N2:3.76'
	#Specify the number of time steps and the time step size
	nt = 100000
	dt = 1.e-6 #s
	Tmin = 0.65
	Tmax = 0.85
	npoints = 11
#Storage
#Temperature storage variables
	
	Ti = np.zeros(npoints,'d')
	Ti2 = np.zeros(npoints,'d')
#The initial storage variable become case dependent
	
	tim = np.zeros(nt,'d')
	temp_cas = np.zeros(nt,'d')
	dtemp_cas = np.zeros(nt-1,'d')
#Additional storage variables are needed to differentiate each case
	
	Autoignition_cas = np.zeros(npoints,'d')
	FinalTemp_cas = np.zeros(npoints,'d')
	mfrac_cas = np.zeros([npoints,gas.n_species],'d')
Note that all storage variables now become storage vectors, storage vectors are now storage matrices,
etc. You can remove the rest of the script, as we will now begin the loop over the several initial
conditions :
#Loop over initial conditions
	
	for j in range(npoints):
		Ti2[j] = Tmin + (Tmax - Tmin)*j/(npoints - 1)
		Ti[j] = 1000/Ti2[j]
		#Set gas state, always at stoichiometry
		gas.TPX = Ti[j], one_atm, 'CH4:0.5,O2:1,N2:3.76'
		#Create the ideal batch reactor
		r =ct.IdealGasReactor(gas)
		# Now create a reactor network consisting of the single batch reactor
		sim = ...
		# Initial simulation time
		time = 0.0
		#Loop for nt time steps of dt seconds.
	for n in range(nt):
		time += dt
		sim.advance(time)
		tim[n] = time
		temp_cas[n] = r.T
		mfrac_cas[j][:] = r.thermo.Y
#################################################################
# Catch the autoignition timings
#################################################################
# Get autoignition timing
	Dtmax = [0,0.0]
	for n in range(nt-1):
		dtemp_cas[n] = (temp_cas[n+1]-temp_cas[n])/dt
		if (dtemp_cas[n]>Dtmax[1]):
		Dtmax[0] = n
		Dtmax[1] = dtemp_cas[n]
# Local print
	Autoignition = (tim[Dtmax[0]]+tim[Dtmax[0] + 1])/2.
	print 'For ' +str(Ti[j]) +', Autoignition time = (s) ' + str(Autoignition)
# Posterity
	Autoignition_cas[j] = Autoignition*1000 #ms
	FinalTemp_cas[j] = temp_cas[nt-1]
#################################################################
# Save results
#################################################################
# write output CSV file for importing into Excel
	csv_file = 'Phi-1_P-1_Trange_UV.csv'
	with open(csv_file, 'w') as outfile:
	writer = csv.writer(outfile)
	writer.writerow(['Auto ignition time','Final Temperature'] + gas.species_names)
	for i in range(npoints):
		writer.writerow(['cas Ti = ', Ti[j]])
		writer.writerow([Autoignition_cas[i], FinalTemp_cas[i]] +
		list(mfrac_cas[i,:]))
		print 'output written to '+csv_file
#################################################################
# Plot results
#################################################################
# create plot
	plot(Ti2,Autoignition_cas, '^', color = 'orange')
	xlabel(r'Temp [1000/K]', fontsize=20)
	ylabel("Autoignition [ms]")
	title(r'Autoignition of $CH_{4}$ + Air mixture at $\Phi$ = 1, and P = 1
	bar',fontsize=22,horizontalalignment='center')
	axis([0.60,0.90,0.0,100.0])
	grid()
#show

	savefig('Phi-1_P-1_Trange_UV.png', bbox_inches='tight')
