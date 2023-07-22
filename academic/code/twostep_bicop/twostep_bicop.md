# How to Conduct `twostep_bicop.do` file

Please follow these steps:

1. get `twostep_bicop.do` file from github
2. Open it in your stata
   > 2.1 This do-file is a temporary file in stata and its function is same as twostop model, thus I do not recommend run it in stata package which may cause recall issues.
   > 2.2 You need add your command at the last line in `twostep_bicop.do`, or it will cause recall issues.
   > 2.3 You can also rename `twostep_bicop.do` to `twostep.do` and replace origial `twostep.do` file, then you can use it by default command, but i do not recommand this way because you need to adjust your conditional variables. Therefore, run in directly is preferable.

3. Command
- bicop command: 
  ```
  bicop (y1=x11 x12) (y2= x21 x22) [iw=weight],copula(frank)
  ```
- twostep_bicop command:
  **conduct model**
  ```
  twostep cohort4: bicop (y1=x11 x12) (y2= x21 x22) [iw=weight] || edv _b_cons cohort4
  ```
  **list all variables**
  ```
  twostep cohort4: bicop (y1=x11 x12) (y2= x21 x22) [iw=weight] || mk2nd _all
  ```

  **copula type**
  Now the default type is `frank`, which has been hard coded.
  if you want to use a new type,you need to go to the `statsby` function,and update a new one

4. Code comments

- 4.1 Get all variables
    I use `gettoken` `parse` `subinstr` function to get all variables and store it in varlist.

    ```
    gettoken equation_1 equation_2: equations, parse(")")	
		local equation_1: subinstr local equation_1 "(" "", all
		local equation_1: subinstr local equation_1 "=" " ", all
		display "equation_1 " "`equation_1'"
    ```

**You could get more information from [stata handbook-gettoken](https://www.stata.com/manuals/pgettoken.pdf)**
**You could get more information from [stata handbook-subinstr](https://www.stata.com/manuals/m-5subinstr.pdf)**



- 4.2 Run `bicop` model with in `statsby` function

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

