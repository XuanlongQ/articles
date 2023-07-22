# How to Conduct `twostep_bicop.do` file

Please follow these steps:

1. Get `twostep_bicop.do` and `bicop.ado` file from github
2. Replace the original `bicop.ado` by the new file
   > use `which(bicop) to find your file path`  
3. Run `twostep_bicop.do` in Stata. <font color='red'> (Recommend) </font>
   > - This do-file is a temporary file in stata and its function is same as twostop model, thus I do not recommend run it in stata package which may cause recall issues.
   > - You could add your command at the last line in `twostep_bicop.do`, or run it in stata directly.
   > - You can also rename `twostep_bicop.do` to `twostep.do` and replace origial `twostep.do` file, then you can use it by default command, But you need to delete `capture` grammer at the head of each function.

4. Command
   
- `twostep_bicop` do file
  
  **4.1 conduct model**
 ```
  twostep cohort4: bicop (y1=x11 x12) (y2= x21 x22) [iw=weight] || edv _b_cons cohort4
 ```
 
  **4.2 list key parameters**
 ```
  twostep cohort4: bicop (y1=x11 x12) (y2= x21 x22) [iw = weight] || mk2nd _all
 ```

- Verify the correctness in `bicop` file
  
  **4.3 bicop command** 
  ```
  bicop (y1=x11 x12) (y2= x21 x22) [iw=weight],copula(frank)
  ```

- Other options
  
  **4.4 copula type**

  Now the default type is `frank`, which has been hard coded.
  if you want to use a new type,you need to go to the `statsby` function,and update a new one
  
  **4.5 weights**
  
  - In `twostep`: unitcpr, fweights, aweights and pweights are allowed;
  - In `bicop`: pweights, fweights, and iweights are allowed;

  And fweights does not support `float` parameters. Thus, the parameter only can be passed is `pweight`.

  However, in `twostep` model, `pweight` does not means `pweight`

  we can see the detail in `twostep`.
  ```
  		// Statsby does not allow pweights. I simulate them with aweights and vce(robust)
		if "`weight'" == "pweight" {
			local weight "aweight"
			//local weight "fweight"
			if "`vce'" == "" {
				local vce vce(robust)
			}
			else {
				local vce `vce' robust
				local vce vce(`: list uniq vce')
			}
			
		}	
  ```

  It means we can not use pweight to pass parameters. Luckily, `iweights` is used for customized the weights. And both `bicop`  could suport this weights.

  Therefore, we just add `iweights` input to `twostep`, then we can use `iweight` passing variables.

5. Code comments

-  Get all variables
    I use `gettoken` `parse` `subinstr` function to get all variables and store it in varlist.

    ```
    gettoken equation_1 equation_2: equations, parse(")")	
		local equation_1: subinstr local equation_1 "(" "", all
		local equation_1: subinstr local equation_1 "=" " ", all
		display "equation_1 " "`equation_1'"
    ```

  > **You could get more information from [stata handbook-gettoken](https://www.stata.com/manuals/pgettoken.pdf)**

  > **You could get more information from [stata handbook-subinstr](https://www.stata.com/manuals/m-5subinstr.pdf)**



- Run `bicop` model with in `statsby` function

    ```
    statsby _b _se _n_model = e(N) `addstats', `clear' by(`byvar') saving(`1stlevelcoefs', double):  ///
		  `model',  `if' `in',  `vce' `noconstant' `hasconstant' `tsconstant'	
    ```

    The first level model's command is:
    ```
    local command (`firstdepvar_e1' = `firstindepvar_e1') (`firstdepvar_e2' = `firstindepvar_e2') [`weight'`exp'] ,copula(frank)
		local model `model' `command'
		display "Command: "  "`model'"
    ```
    You can add to delete variable in this model.

**You could get more information from [stata handbook-statsby](https://www.stata.com/manuals/dstatsby.pdf)**

6. Test the correctness

- Test the weights
  In `bicop`:
  test the result between `iweight` and `pweight`
  ```
  1. bicop (y1=x11 x12) (y2= x21 x22) [iw=weight],copula(frank)
  2. bicop (y1=x11 x12) (y2= x21 x22) [pw=weight],copula(frank)
  ```
  The results are same, thus we could know the `iweight` has converted to `pweight`

- Test the se and coeffecients
  In `bicop`:
  ```
  bicop (y1=x11 x12) (y2= x21 x22) if cohort4 == 1 [pw=weight],copula(frank)
  ```

  the result shows:
  - _se_cons: .14833392
  - _b_cons: 4.441093

- Test the consistence with `twostep` and `bicop`
  
  In `twostep`:
  ```
  twostep cohort4: bicop (y1=x11 x12) (y2= x21 x22) [iw = weight] || mk2nd _all
  list _se_cons _b_cons
  ```

  The result shows:
  - _se_cons: .14833392
  - _b_cons: 4.441093