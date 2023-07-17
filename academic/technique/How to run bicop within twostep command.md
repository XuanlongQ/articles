# How to run bicop within twostep command

## What is twostep in Stata ?
twostep is a bundle of programs to ease multilevel analyses with the twostep approach. The twostep approach to mulitlevel analysis means to separately estimate a parameter of interest in a unit level data set (e.g. individuals) for all categories of a cluster level variable (e.g. countries, states, schools, etc.). The twostep approach is sometimes seen as superior to the more standard one-step approach (see mixed) if the numbers of observation on the second level becomes small (see Achen 2005). Additionally, two-step mulitlevel analysis may be used for checking the model assumptions of the one-step approach, or for exploratory data analysis of multilevel data.

## How I run a first level with own command in twostep?

Twostep's basic syntax is 

```
twostep cohort4: reg y1 x1 x2  || edv _b_cons cohort4
```
- twostep means the model we use

- cohort4 means the level-2 variables

- reg means the first level regression model

- y1 is dependent variable

- x1 is the first independent variable

- x2 is the second independent variable

so we can we run a basic regression model at level-1 and use cohort4 as my level-2 model

However,

**What should I do If my fisrt level model have two equation?**

we might this command
```
twostep cohort4: `model' y1 x11 x12 y2 x21 x22   || edv _b_cons cohort4
```

For simply the question, we ingore the specific model. Then, if you conduct your command, you will find

- y1 is assigned to dependent variable

- x11 x12 y2 x21 x22 are assigend to independent variables in a varlist.

It obviously can not match our requirements.

### Why?

Let us go to the source code

```ado
syntax varlist(fv) [fweight aweight pweight] [if] [in] [, vce(string) NOCONStant Hascons tsscons eform(string) clear]
gettoken firstdepvar firstindepvar: varlist
```
Yes, the author use gettoken get first variable and put other variables to varlist.

Then the author use `statsby` command:

Like,
```ado
statsby _b _se _n_model = e(N) `addstats', `clear' by(`byvar') saving(`1stlevelcoefs', double):  ///
`model' [`weight'`exp'] `if' `in',  `vce' `noconstant' `hasconstant' `tsconstant'
```

Thus ,we can see if the `model` is `reg` it will conduct regression model at first level.

### How to fix our problems?  Two equations at the first level ?
Easy job,
I get all variables by iterating get varlist, More specifically,
```
gettoken firstdepvar rest: varlist
display "firstdepvar is, " "`firstdepvar'"
		
gettoken firstindepvar1 rest: rest
gettoken firstindepvar2 rest: rest

display "firstindepvar is, " "`firstindepvar'"
		
	
		
gettoken firstdepvar_e2 rest: rest
display "firstdepvar_e2 is, " "`firstdepvar_e2'"
		
		
gettoken firstdepvar_e2_indep1 rest: rest
gettoken firstdepvar_e2_indep2 rest: rest
		
display "firstindepvar_e2 is, " "`firstindepvar_e2'"
```
Then, I get every single variable.

Afterwards, I use this variable to merge the `model` parameter
like 
```
local model `model' (`firstdepvar' = `firstindepvar1' `firstindepvar2') (`firstdepvar_e2' = `firstdepvar_e2_indep1' `firstdepvar_e2_indep2')
```

then we could pass `model` to the `statsby` function.

### Summary
Based on this, we could run all first level model to second level by twostep command.

I will use bicop as a eexample.
#### step1
```

if "`model'" == "bicop" {
		// Only in bicop model that this function will work.
		
		syntax varlist(fv) [fweight aweight pweight] [if] [in] [, vce(string) NOCONStant Hascons tsscons eform(string) clear]
		
		
		gettoken firstdepvar rest: varlist
		display "firstdepvar is, " "`firstdepvar'"
		
		gettoken firstindepvar1 rest: rest
		gettoken firstindepvar2 rest: rest
		local firstindepvar `firstindepvar1' `firstindepvar2'
		display "firstindepvar is, " "`firstindepvar'"
		
	
		
		gettoken firstdepvar_e2 rest: rest
		display "firstdepvar_e2 is, " "`firstdepvar_e2'"
		
		
		gettoken firstdepvar_e2_indep1 rest: rest
		gettoken firstdepvar_e2_indep2 rest: rest
		local firstindepvar_e2 `firstdepvar_e2_indep1' `firstdepvar_e2_indep2'
		display "firstindepvar_e2 is, " "`firstindepvar_e2'"
	}
```
#### step2
```
statsby _b _se _n_model = e(N) `addstats', `clear' by(`byvar') saving(`1stlevelcoefs', double):  ///
		  `model' (`firstdepvar' = `firstindepvar1' `firstindepvar2') (`firstdepvar_e2' = `firstdepvar_e2_indep1' `firstdepvar_e2_indep2'), copula(frank), [`weight'`exp'] `if' `in',  `vce' `noconstant' `hasconstant' `tsconstant'
```

Hope it can help you !

## References
1. [The introduction of twostep's pdf - click ](https://www.stata.com/meeting/germany21/slides/Germany21_Giesecke.pdf)
2. [Simple introduction of two step](https://econpapers.repec.org/software/bocbocode/s459021.htm)
