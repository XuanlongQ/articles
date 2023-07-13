# Comments of "bicop.ado" file.md

## Proggram Replay
```
program Replay, eclass
	syntax [, Level(cilevel)]
```
This line sets up the syntax for the Estimate command, allowing the user to specify the confidence level for confidence intervals using the option Level(cilevel).
```
	local copula `e(copula)'
	local mixture `e(mixture)'
```
These lines store the values of the system variables e(copula) and e(mixture) in local macros copula and mixture, respectively. These variables contain information about the specified copula type and mixture option.
```
	if "`copula'" == "indep" & "`mixture'" == "none" {
		ml display, level(`level') nolstretch
	}
	else {
		ml display, level(`level') nolstretch plus 
	}
```
These lines display the output of the estimation. If the copula type is "indep" and the mixture option is "none", it displays the estimation results without any additional information. Otherwise, it displays the estimation results with additional information using the plus option.
```	
	//	Set up diparm for dependency parameter to be reported in natural form
	local temp = -invnormal((1-(`level'/100))/2) // use level to calculate z value for ci
	
	if "`copula'" == "gaussian" {
		_diparm depend, tanh prob notab
		 output_line "theta" `r(est)' `r(se)' `r(z)' `r(p)' `temp'		 
	}
    if "`copula'" == "frank" {
       	_diparm depend, f(@) derivative(1) prob notab
		output_line "theta" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
	}
    if "`copula'" == "clayton" {
      	_diparm depend, f(exp(@)) derivative(exp(@)) prob notab 
		output_line "theta" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
	}
    if "`copula'" == "joe"|"`copula'" == "gumbel" {
       	_diparm depend, f(exp(@)+1) derivative(exp(@)) prob notab
		output_line "theta" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
	}


	if "`copula'" != "indep" {	
		matrix theta = r(est)
		mat colnames theta = theta
	}
	
	if "`copula'" == "indep" {
		matrix extpar = 1
	}
	else {
		matrix extpar = theta
	}
	
	// set up diparm for mixture parameters to be reported in natural form
	
	if "`mixture'" == "both" | "`mixture'" == "mix1" | "`mixture'" == "equal" {
		_diparm pu1, invlogit prob notab
		output_line "pi_u_1" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix u = r(est)
		_diparm pu1, func(1/(1+exp(@))) der(-exp(@)/((1+exp(@))^2)) prob notab
		output_line "pi_u_2" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix u = u , r(est)
		_diparm pu1 mu2, func(-(@2)/exp(@1)) der(@2/exp(@1) (-1/exp(@1))) prob notab
		output_line "mean_u_1" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix u = u , r(est)
		_diparm mu2, func(@) der(1) prob notab
		output_line "mean_u_2" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix u = u , r(est)
		_diparm pu1 mu2 su2, func((1+exp(@1)-((@3^2)+(@2^2)))/exp(@1)-(-@2/exp(@1))^2) der(1-(1+exp(@1)-((@3^2)+(@2^2)))/exp(@1)+2*(@2/exp(@1))^2 ((@2/exp(@1))*(-2-exp(-@1))) (-2*@3/exp(@1))) prob notab			
		output_line "var_u_1" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix u = u , r(est)
		_diparm su2, func(@^2) der(2*@) prob notab
		output_line "var_u_2" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix u = u , r(est)
		mat colnames u = pi_u_1 pi_u_2 mean_u_1 mean_u_2 var_u_1 var_u_2
		matrix extpar = extpar,u
	}
		
	if "`mixture'" == "both" | "`mixture'" == "mix2" {
		_diparm pv1, invlogit prob notab
		output_line "pi_v_1" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix v = r(est)
		_diparm pv1, func(1/(1+exp(@))) der(-exp(@)/((1+exp(@))^2)) prob notab
		output_line "pi_v_2" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix v = v , r(est)
		_diparm pv1 mv2, func(-(@2)/exp(@1)) der(@2/exp(@1) (-1/exp(@1)))prob notab
		output_line "mean_v_1" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix v = v , r(est)
		_diparm mv2, func(@) der(1) prob notab
		output_line "mean_v_2" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix v = v , r(est)
		_diparm pv1 mv2 sv2, func((1+exp(@1)-((@3^2)+(@2^2)))/exp(@1)-(-@2/exp(@1))^2) der(1-(1+exp(@1)-((@3^2)+(@2^2)))/exp(@1)+2*(@2/exp(@1))^2 ((@2/exp(@1))*(-2-exp(-@1))) (-2*@3/exp(@1)))prob notab
		output_line "var_v_1" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix v = v , r(est)
		_diparm sv2, func(@^2) der(2*@) prob notab
		output_line "var_v_2" `r(est)' `r(se)' `r(z)' `r(p)' `temp'
		matrix v = v , r(est)
		mat colnames v = pi_v_1 pi_v_2 mean_v_1 mean_v_2 var_v_1 var_v_2
		matrix extpar = extpar,v
	}
	
	if  "`copula'" != "indep" {
		ereturn matrix extpar extpar
		di as text "{hline 13}{c BT}{hline 64}"
	} 
	
	else if  "`copula'" == "indep"  & colsof(extpar) > 1  {
		matrix extpar = extpar[1, 2.. colsof(extpar) ]
		ereturn matrix extpar extpar	
		di as text "{hline 13}{c BT}{hline 64}"
	}
	
	
	
	
	if "`diparm'" != "" {
		matrix auxpar=r(table)
		matrix auxpar = auxpar[1, "_diparm1:"]
		matrix colnames auxpar = _:
		ereturn matrix auxpar auxpar
	}
	
	if `"$bc_waldtest"' != ""  {
		display `"$bc_waldtest"'
	}
	
	if `"$bc_waldindp"' != ""  {
		display `"$bc_waldindp"'
	}
end
```


## Proggram Estimate
This section of the Stata program defines a program called `Estimate` with the options `eclass` and `sortpreserve`. Here is a line-by-line description of the code:

```
program Estimate, eclass sortpreserve
```
This line defines a program called `Estimate` with the options `eclass` and `sortpreserve`. The `eclass` option allows the program to return estimation results as e-class matrices, and the `sortpreserve` option ensures that the order of the variables in the dataset is preserved.

```
	local cmdline `"bicop `0'"'
```
This line sets up a local macro called `cmdline` with the value of the string `"bicop `0'"'. This string contains the name of the estimation command to be run (`bicop`) and any options specified by the user (`0`).

```
foreach global in offo1 offo2 Nthr1 Nthr2 copu mix kx1 kx2 waldtest waldindp {
	macro drop bc_`global'
}
```
These lines drop any global macros that will be used in the estimation. The macro names are constructed using the prefix `bc_` and the names of the variables to be dropped (`offo1`, `offo2`, `Nthr1`, `Nthr2`, `copu`, `mix`, `kx1`, `kx2`, `waldtest`, and `waldindp`).


```
/* syntax taken from biprobit and modified*/
/* two syntax, handle one at a time */

	gettoken first : 0, match(paren)	
	if "`paren'" == "" {
	
/* syntax 1, bivariate model with common covariate vector 
										- parenthesis has not been found */
										
		gettoken dep1 0:0, parse (" =,[")
		_fv_check_depvar `dep1' /* check that dep var isn't a factor variable */
		tsunab dep1 : `dep1' /* Expand a variable with time series operators */
		rmTS `dep1' /* deals with time series operators in the dependent variable*/
		confirm variable `r(rmTS)'
		gettoken dep2 0:0, parse (" =,[")
		_fv_check_depvar `dep2'
		tsunab dep2 : `dep2'
		rmTS `dep2'
		confirm variable `r(rmTS)'
		gettoken junk left :0, parse ("=")
		if "`junk'" == "=" {
			local 0 "`left'"
		}
		syntax [varlist(default=none ts fv)] [if] [in] [pw fw iw] /*
			*/[, COPula(string) MIXture(string) /* 
			*/ Robust CLuster(varname)   	/*
			*/ offset1(varname) offset2(varname)    /*
			*/ Level(cilevel) 		/* 
			*/ noLog FROM(string) SEarch(string) Repeat(string)		/*
			*/ /*ITERate(passthru)*/ VCE(passthru) Constraints(string) * ]
		
		local ind1 `varlist'
		local ind2 `varlist'

		
        local dep1n "`dep1'"
        local dep1n : subinstr local dep1 "." "_"
        local dep2n "`dep2'"
        local dep2n : subinstr local dep2 "." "_"

        local e_neq = 2
		
		//mlopts mlopts, `options'
		local option0 `options'
		marksample touse
		markout `touse' `dep1' `dep2' `offset1' `offset2', strok
	
	}
	else {
// syntax 2, different covariate vectors - with parentheses
// get first equation 
		gettoken first 0:0, parse(" ,[") match(paren)
		local left "`0'"
		local junk: subinstr local first ":" ":", count(local number) /* This 
											picks up equation name if given */
		if "`number'" == "1" {
			gettoken dep1n first: first, parse(":")
			gettoken junk first: first, parse(":")
		}
		local first : subinstr local first "=" " "
		gettoken dep1 0: first, parse(" ,[") 
		_fv_check_depvar `dep1'
		tsunab dep1: `dep1'
		rmTS `dep1' 
		confirm variable `r(rmTS)'
		if "`dep1n'" == "" {
			local dep1n "`dep1'"
		}
		syntax [varlist(default=none ts fv)] [, OFFset(varname numeric)]
		local ind1 `varlist'
		local offset1 `offset' 
		
// get second equation 
		local 0 "`left'"
		gettoken second 0:0, parse(" ,[") match(paren)
		if "`paren'" != "(" {
			dis in red "two equations required"
			exit 110
		}
		local left "`0'"
		local junk : subinstr local second ":" ":", count(local number)
		if "`number'" == "1" {
			gettoken dep2n second: second, parse(":")
			gettoken junk second: second, parse(":")
		}
		local second : subinstr local second "=" " "
		gettoken dep2 0: second, parse(" ,[") 
		_fv_check_depvar `dep2'
		tsunab dep2: `dep2' 
		rmTS `dep2'
		confirm variable `r(rmTS)'
		if "`dep2n'" == "" {
			local dep2n "`dep2'"
		}
		syntax [varlist(default=none ts fv)] [, OFFset(varname numeric) ]
		local ind2 `varlist'
		local offset2 `offset' 
```
This is a Stata program that defines a command called Estimate. When this command is executed, it will estimate a bivariate probit model using the bicop command and return the results in an eclass.

The program first sets a local macro called cmdline to be equal to the string "bicop 0'", where 0 is the first argument passed to theEstimatecommand. This string is then used to call the `bicop` command with the specified time-series variable name.

Next, the program drops several global variables that will be used later in the program. It then checks whether the input syntax for the Estimate command matches one of two possible formats. 

If it matches the first format (which involves a single covariate vector), the program extracts the names of the dependent variables for each equation and stores them in local macros called dep1 and dep2. It also checks that these variables are valid Stata variables using the confirm variable command.

If the input syntax matches the second format (which involves two sets of covariate vectors enclosed in parentheses), the program extracts the names of the dependent variables for each equation and stores them in local macros called dep1 and dep2. It also checks that these variables are valid Stata variables using the confirm variable command.

Finally, the program calls the bicop command using the specified syntax, passing in the names of the dependent variables and covariate vectors, as well as any specified offset variables. The results of the estimation are stored in an eclass, which can be accessed using various Stata commands.



```
// remaining options 
		local 0 "`left'" // This line sets up a local macro called `0` with the value of the local macro `left`.
		syntax [if] [in] [pw fw iw] [, COPula(string) MIXture(string)   /*
			*/ Robust Cluster(varname) /*ITERate(passthru)*/ /*
			*/ Level(cilevel)  	   /*
			*/ noLOG FROM(string) SEarch(string) Repeat(string) VCE(passthru) * ]

```
This line sets up the syntax for the remaining options of the Estimate command. These options include COPula, which specifies the copula type, MIXture, which specifies the mixture option, Robust, which specifies robust standard errors, Cluster, which specifies clustering of standard errors, Level, which specifies the confidence level for confidence intervals, noLOG, which suppresses logging of the command, FROM, which specifies starting values for the estimation, SEarch, which specifies the search method for finding starting values, Repeat, which specifies the number of times to repeat the estimation, and VCE, which specifies the estimator for the variance-covariance matrix.


```
		local option0 `options'  // This line sets up a local macro called option0 with the value of the system variable options, which contains all options specified by the user.

		marksample touse // These lines mark the sample and variables to be used in the estimation. The sample is marked as touse, and the dependent variables, independent variables, and offset variables are marked as well. This allows these variables to be used in subsequent estimation commands.

		markout `touse' `dep1' `dep2' `ind1' `ind2' `offset1'  `offset2'
	}
```

```
//	Various options...	
	local wtype `weight'
    local wtexp `"`exp'"'
    if "`weight'" != "" { 
		local wgt `"[`weight'`exp']"'  
	} // These lines set up local macros for the weight variable and exponent used in the estimation. If a weight variable is specified, it is stored in the macro wgt.

	_get_diopts diopts option0, `option0'
	mlopts mlopts, `option0' // These lines set up options for diagonal parameterization estimation. The _get_diopts command sets up options for diagonal parameterization, and mlopts sets up options for maximum likelihood estimation.


	local cns `s(constraints)'	
	if "`cluster'" ! = "" { 
		local clopt "cluster(`cluster')" 
	} // These lines set up options for constraints and clustering. The local macro cns is set to the value of the system variable s(constraints), which contains any constraints specified by the user. If a clustering variable is specified, it is stored in the macro clopt.


	_vce_parse, argopt(CLuster) opt(Robust oim opg) old: ///
                [`weight'`exp'], `vce' `clopt' `robust'	
        local cluster `r(cluster)'
        local robust `r(robust)' // These lines set up options for variance-covariance matrix estimation. The _vce_parse command parses the options for clustering and robust standard errors, and sets the local macros cluster and robust to their respective values.


	if "`cluster'" ! = "" { 
		local clopt "cluster(`cluster')" 
	} // This line sets up the clustering option again, in case it was not specified earlier.


	local cns `s(constraints)'

	if `"`robust'"' != "" {
		local crtype crittype("log pseudolikelihood")
	} // These lines set up options for robust standard errors. If robust standard errors are specified, the local macro crtype is set to "log pseudolikelihood".
```

```
//	Check validity of copula declaration & set up global for copula name
		capture assert 	"`copula'" == "" | "`copula'" == "gaussian"  /*
			*/ |"`copula'" == "frank" |"`copula'" == "clayton"  /*
			*/ | "`copula'" == "joe"|"`copula'" == "gumbel" |"`copula'" == "indep" 
		if _rc == 9 {
			di in red "Copula must be indep, gaussian, frank, clayton, gumbel or joe"
			exit 198
		}
		if "`copula'" == "" {
			local copula "gaussian"
		}	
		global bc_copu = "`copula'"

//	Check validity of mixture option & set up global for mixture choice
		capture assert 	"`mixture'" == "" | "`mixture'" == "none" |  "`mixture'" == "equal" | /*
		*/ "`mixture'" == "mix1" | "`mixture'" == "mix2" | "`mixture'" == "both" 
		if _rc == 9 {
			di in red "Mixture must be none, mix1, mix2, equal or both"
			exit 198
		}
		if "`mixture'" == "" {
			local mixture "none"			
		}	
		global bc_mix = "`mixture'"	 
	}


// handling from	
	if "`from'" == "" {
		tempname a from0 mixname
//	Ordered probits to supply starting values
/*
		noi display in green "*************************************************"
		noi display in green "Univariate ordered probits for starting values"
		noi display in green "*************************************************"
*/
		qui oprobit `dep1' `ind1' if `touse' `wgt', `offo1' /* // add quietly
			*/ iter(`=min(1000,c(maxiter))') /*`mlopts'*/ constraints(`constraints')
			
		scalar ll_ind=e(ll)
		if _rc == 0 {
			tempname cb1 ct1 tmp 
			global bc_kx1=e(k)-e(k_aux)
			mat `tmp' = e(b)
			if `"`ind1'"' != ""  {
				mat `cb1'=`tmp'[1,"`dep1':"] // ml model does not allow
					//to pass an equation with ,nocons and no variables 
					// so the model with no variables needs to be handled
					// separately
			}
			
			local nthr1=$bc_Nthr1
			mat `ct1'=`tmp'[1,"cut1:_cons".."cut`nthr1':_cons"]
			local cuts
			forvalues k=1/`nthr1' {
				local cuts "`cuts' /cuteq1_`k'"
				local ceqs "`ceqs' cuteq1_`k'"
			}
			mat coleq `ct1'=`ceqs' 
			mat colnames `ct1'=_cons
		}	
				
		qui oprobit `dep2' `ind2' if `touse' `wgt', `offo2' /*  
			*/ iter(`=min(1000,c(maxiter))') /*`mlopts'*/ constraints(`constraints')
		scalar ll_ind=ll_ind+e(ll)
		if _rc == 0 {
			tempname cb2 ct2 tmp
			global bc_kx2=e(k)-e(k_aux)
			mat `tmp' = e(b)
			if `"`ind2'"' != ""  {
				mat `cb2'=`tmp'[1,"`dep2':"]
			}	
			local nthr2=$bc_Nthr2
			mat `ct2'=`tmp'[1,"cut1:_cons".."cut`nthr2':_cons"]
			local cuts 
			local ceqs
			forvalues k=1/`nthr2' {
				local cuts "`cuts' /cuteq2_`k'"
				local ceqs "`ceqs' cuteq2_`k'"
			}
			mat coleq `ct2'=`ceqs'
			mat colnames `ct2'=_cons
		}
		global ll_ind=ll_ind
		noi di "LogL for independent ordered probit model " ll_ind  
		
//	Insert into vector of start values

		if "`ind1'" != "" & "`ind2'" != "" {
			//mat `from0' = `cb1',`cb2',`ct1',`ct2'
			mat `from0' = `cb1',`cb2', `ct1',`ct2'
		}
		else if ("`ind1'" == "") & ("`ind2'" == "") {
			mat `from0' = `ct1',`ct2'
		}
		else if (("`ind1'" != "") & ("`ind2'" == ""))  { 
			mat `from0' = `cb1', `ct1',`ct2'
		}
		else if (("`ind1'" == "") & ("`ind2'" != "")) { 
			mat `from0' = `cb2', `ct1',`ct2'
			
		}
		
	
		
		//	Starting value for transformed dependency parameter if not independent
		if "`copula'" != "indep" {
			mat `a' = (1.2)
			mat colnames `a' = depend:_cons
			mat `from0' = `from0',`a'
		}
		if "`mixture'" == "equal" | "`mixture'" == "mix1" {
			mat `mixname'=-0.01,-0.01,1.01
			mat colnames `mixname' = pu1:_cons mu2:_cons su2:_cons
			mat `from0'=`from0',`mixname'
		}	
 		if "`mixture'" == "mix2" {
			mat `mixname'=-0.01,-0.01,1.01
			mat colnames `mixname' = pv1:_cons mv2:_cons sv2:_cons
			mat `from0'=`from0',`mixname'
		}	
      	if "`mixture'" == "both" {	
			mat `mixname'=0.8,-0.35,1.4,0.8,-0.35,1.4
			mat colnames `mixname' = pu1:_cons mu2:_cons su2:_cons pv1:_cons /// 
									mv2:_cons sv2:_cons
			mat `from0'=`from0',`mixname'
		}
		local from="`from0'"
		mat `from'=`from0'
		
	}

	else {
		local nn : word count `ind1'
		global bc_kx1= `nn'
		local nn : word count `ind2'
		global bc_kx2= `nn'
		
		/*running oprobits to get ll_ind and starting values if needed*/
		qui oprobit `dep1' `ind1' if `touse' `wgt', `offo1' /* // add quietly
			*/ iter(`=min(1000,c(maxiter))') /*`mlopts'*/ constraints(`constraints')
		scalar ll_ind=e(ll)
		qui oprobit `dep2' `ind2' if `touse' `wgt', `offo2' /*  // add quietly
			*/ iter(`=min(1000,c(maxiter))') /*`mlopts'*/ constraints(`constraints')
		scalar ll_ind=ll_ind+e(ll)
		global ll_ind=ll_ind
		noi di "LogL for independent ordered probit model " ll_ind  
		
	}"
```
This section of the Stata program checks the validity of the copula and mixture options passed to the Estimate command and sets up global macros for their values. If an invalid option is passed, the program returns an error message and exits. If no option is passed, default values are set.

If no from option is passed to the Estimate command, the program generates starting values for the estimation using univariate ordered probits. It first sets up a temporary matrix to hold the starting values and a temporary name for it. The program then runs two univariate ordered probits on the dependent variables and independent variables specified in dep1, ind1, dep2, and ind2, using any weight variables and offset variables specified in the command. It stores the log-likelihood from these models in a global macro called ll_ind.

If both ind1 and ind2 are specified, the program stores the coefficients and cutpoints from both models in separate matrices. If only one independent variable is specified, it stores only that variable's coefficients and cutpoints. If no independent variables are specified, it stores only the cutpoints.

The program then concatenates these matrices into a single matrix called from0. If a copula other than "indep" is specified, it adds a temporary matrix called a containing starting values for the transformed dependency parameter to from0. If a mixture option is specified, it adds a matrix containing starting values for the mixture parameters to from0.

If a from option is passed to the Estimate command, the program parses the independent variables specified in ind1 and ind2, stores their count in global macros called bc_kx1 and bc_kx2, and runs univariate ordered probits on them to get starting values and log-likelihoods. It then concatenates these starting values with any mixture or copula parameters specified in the from option into a matrix called from.

```
//	***  PARAMETER NAMES  ***

//	threshold parameters
	forvalues  i = 1(1)$bc_Nthr1 {
		local thr10 `"`thr10' /cuteq1_`i'"'
	}
	forvalues  i = 1(1)$bc_Nthr2 {
		local thr20 `"`thr20' /cuteq2_`i'"'
	}



//	coefficient names

	if "`ind1'"!="" & "`ind2'"!="" {
		local equs0 `"`equs0' (`dep1n': `y1' = `ind1', `offo1'  nocons  )"'
	    local equs0 `"`equs0' (`dep2n': `y2' = `ind2', `offo2'  nocons  )"'
		local equs0 `"`equs0' `thr10' `thr20'"'
		
	}
	else if "`ind1'"=="" & "`ind2'"=="" {
		local equs0 `"`equs0' (cuteq1_1: `y1' = , `offo1'  )"'
		gettoken drop thr10 : thr10
		local equs0 `"`equs0' `thr10'"'
		local equs0 `"`equs0' (cuteq2_1: `y2' = , `offo2'  )"'
		gettoken drop thr20 : thr20
		local equs0 `"`equs0' `thr20'"'
	}
	else if ("`ind1'"=="") & ("`ind2'"!="") {
		local equs0 `"`equs0' (`dep2n': `y2' = `ind2', `offo2'  nocons  )"'
		local equs0 `"`equs0' (cuteq1_1: `y1' = , `offo1'  )"'
		gettoken drop thr10 : thr10
		local equs0 `"`equs0' `thr10' `thr20'"'
	}
	else if ("`ind1'"!="") & ("`ind2'"=="") {
		local equs0 `"`equs0' (`dep1n': `y1' = `ind1', `offo1'  nocons  )"'
		local equs0 `"`equs0' `thr10'"'
	    local equs0 `"`equs0' (cuteq2_1: `y2' = , `offo2'  )"'
		gettoken drop thr20 : thr20
		local equs0 `"`equs0' `thr20'"'
	}
	
	
	
	if "`copula'" != "indep" {
		local equs0 `"`equs0'  /depend"'	/* transformed dependency parameter
										   is always called "depend" */
	}
	
	
//	mixture parameters

       if "`mixture'" == "equal" | "`mixture'" == "mix1" {	
		local equs0 `"`equs0'  /pu1 /mu2 /su2"'
	}	
       if "`mixture'" == "mix2" {	
		local equs0 `"`equs0'  /pv1 /mv2 /sv2"'
	}	
	if "`mixture'" == "both" {	
		local equs0 `"`equs0'  /pu1 /mu2 /su2 /pv1 /mv2 /sv2 "'
	}	
	


//	Call ML here, waldtest restricts all coefficients in the first two equations to zero
		`log' ml model lf1 bicop_lf1 `equs0'	if `touse' `wgt', /// 
				noconstant collinear missing maximize difficult `robust' `mlopts' ///
				init(`"`from'"') search(`search') repeat(`repeat') constraints(`constraints') `clopt' waldtest(-2) //gradient
			
// Wald test of common coefficients
	capture test[`dep1n' = `dep2n'], common	
	if _rc == 0 {   // there are common variables
		local wdf = r(df)
		local wchi2 = string(r(chi2),"%10.3f")
		local wp = string(r(p),"%4.3f")
		global bc_waldtest = `"Wald test of equality of coefficients chi2(df = `wdf')= `wchi2' [p-value=`wp']"'
	}
	else {
		global bc_waldtest = ""    // there are no variables in common			
	}
	
// Wald test of independence for Gaussian, Clayton and Frank
	quietly {
		if "$bc_copu"=="gaussian" {
			capture testnl tanh([depend]:_cons) = 0
			if _rc == 0 { 	
				scalar wdfi = r(df)
				scalar wchi2i = round(r(chi2),.001)
				scalar wpi = round(r(p),.001)
				local wdfib = r(df)
				local wchi2ib = string(r(chi2),"%10.3f")
				local wpib = string(r(p),"%4.3f")
				global bc_waldindp = `"Wald test of independence chi2(df = `wdfib')= `wchi2ib' [p-value=`wpib']"'
				ereturn scalar df_ind = wdfi 
				ereturn scalar chi2_ind = wchi2i
				ereturn scalar p_ind = wpi
			}
			else {
				global bc_waldindp = ""
			}
		}
		else if "$bc_copu"=="frank" {
			capture test [depend]:_cons
			if _rc == 0 {
				scalar wdfi = r(df)
				scalar wchi2i = round(r(chi2),.001)
				scalar wpi = round(r(p),.001)
				local wdfib = r(df)
				local wchi2ib = string(r(chi2),"%10.3f")
				local wpib = string(r(p),"%4.3f")
				global bc_waldindp = `"Wald test of independence chi2(df = `wdfib')= `wchi2ib' [p-value=`wpib']"'
				ereturn scalar df_ind = wdfi 
				ereturn scalar chi2_ind = wchi2i
				ereturn scalar p_ind = wpi
			}
			else {
				global bc_waldindp = ""
			}	
		
		} 
		else if "$bc_copu"=="clayton" {
			capture testnl exp([depend]:_cons) = 0
			if _rc == 0 {
				scalar wdfi = r(df)
				scalar wchi2i = round(r(chi2),.001)
				scalar wpi = round(r(p),.001)
				local wdfib = r(df)
				local wchi2ib = string(r(chi2),"%10.3f")
				local wpib = string(r(p),"%4.3f")
				global bc_waldindp = `"Wald test of independence chi2(df = `wdfib')= `wchi2ib' [p-value=`wpib']"'
				ereturn scalar df_ind = wdfi 
				ereturn scalar chi2_ind = wchi2i
				ereturn scalar p_ind = wpi
			}
			else {
				global bc_waldindp = ""
			}	
		} 
		else {
			global bc_waldindp = ""    			
		}
	}
```
This section of the Stata program defines the parameter names for use in the Estimate command. It creates local macros for the threshold parameters and coefficient names based on the independent variables specified in the command. If both independent variables are specified, it creates two equations with their respective dependent variables, independent variables, and threshold parameters. If only one independent variable is specified, it creates one equation with its dependent variable, independent variable, and threshold parameters. If no independent variables are specified, it creates two equations with their respective dependent variables and threshold parameters.

If a copula other than "indep" is specified, the program adds a transformed dependency parameter to the coefficient names. If a mixture option is specified, it adds mixture parameters to the coefficient names.

The program then calls the maximum likelihood estimation (MLE) command ml with the specified options and parameter names. It also performs a Wald test of equality of coefficients and a Wald test of independence for Gaussian, Clayton, and Frank copulas. The results of these tests are stored in global macros.

```
/************************************************************************************/
	// Return extra results
	ereturn local cmd bicop
	ereturn local cmdline `cmdline'
	ereturn local copula `copula'
	ereturn local mixture `mixture'	
	ereturn local depvar "`dep1' `dep2'"	
	ereturn local indepvars1 `ind1'	
	ereturn local indepvars2 `ind2'	
	local title /// 
			"Generalized bivariate ordinal regression model (copula: `copula', mixture: `mixture')"
	ereturn local title `title'
	
	capture confirm variable bc_offo1
	if !_rc {
		ereturn local offset1 `offset1'
		qui drop bc_offo1 
	}
	capture confirm variable bc_offo2
	if !_rc {
		ereturn local offset2 `offset2'
		qui drop bc_offo2 
	}
	
	ereturn local predict "bicop_p"  
	
	
	if "$bc_copu"=="indep" {
		local adjind = 1
	}
	else {
		local adjind = 0
	}


	if "`mixture'" == "none" { 
		ereturn scalar k_aux = $bc_Nthr1 +$bc_Nthr2+ 1 -`adjind'
	}
    if "`mixture'" == "equal" |"`mixture'" == "mix1" | "`mixture'" == "mix2" {	
		ereturn scalar k_aux = $bc_Nthr1 +$bc_Nthr2+ 1 + 3 - `adjind'
	}	
    if "`mixture'" == "both" {	
		ereturn scalar k_aux = $bc_Nthr1 +$bc_Nthr2+ 1 + 6 - `adjind'
	}	
	
	ereturn scalar ll_c = $ll_ind  //loglikelihood for the comparison model
	Replay, `level'
end"
```
This section of the Stata program returns extra results from the Estimate command. It creates local macros for the command used, copula type, mixture option, dependent variables, independent variables, and title, and stores them in the ereturn command. If an offset variable is specified in the command, it stores its value in a local macro called offset1 or offset2, depending on which dependent variable it corresponds to.

The program then sets the name of the predicted variable to "bicop_p" and determines the number of parameters in the model based on the number of threshold parameters, copula type, and mixture option. It stores this value in a scalar called k_aux.

Finally, the program calculates the log-likelihood for the comparison model (i.e. univariate ordered probit models on each dependent variable) and stores it in a scalar called ll_c. It then replays the estimation command at the specified level of detail.
