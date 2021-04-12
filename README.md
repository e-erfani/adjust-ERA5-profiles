# adjust-ERA5-profiles

This code sharpens ERA5 temperature and moisture profiles near the inversion level.


Main approach:

· Inputs: Tl_era = T_era + g*z/Cp - (L/Cp)*qc, qt_era (= qv+qc), Microwave LWP, ERA Zinv. Note that Tl is the liquid-water static energy divided by Cp, and
          qt is total water mixing ratio.

· Choose inversion height, zi: compute inversion height based on ERA profiles using min of d(RH)/dz*d(THETAL)/dz. 
                               Alternatively, use MODIS CTH.

· Build Tl(z) and qt(z) in two parts: FT and BL


· First FT: 

   o Let L_FT be some height above the inversion where ERA profiles don't "feel" the BL anymore, say 500m. 
   
   o For z>zi+L_FT, Tl(z) = Tl_era(z).  Similar for qt(z)
   
   o For zi < z < zi+L_FT, fit a line to the ERA Tl/qt profiles away from the inversion, and extrapolate down to the inversion.  
     In matlab, this would be Tl(zind2) = polyval( polyfit(z(zind),Tl_era(zind),1), z(zind2) ) where zind are the indices where zi+L_FT < z < zi+3*L_FT 
     and zind2 has zi<z<zi+L_FT.  
   
   o Note: You might need to blend smoothly across zi+L_FT to avoid discontinuities.  
   
   o The top of the region where you're fitting the line is also arbitrary.  I chose  zi+3*L_FT as a reasonable first guess.


· Within the BL

  o The whole temperature and moisture profiles within the BL are:
  
  ![image](https://user-images.githubusercontent.com/28571068/114353870-5f992f00-9b22-11eb-971e-3b4117bb60bd.png)


  o Similarly, qt(z) = max( qt_era(z), qt_ML)


· How to solve:

o Make a function in python/matlab that:

§ takes inputs: Tl_era(z), qt_era(z), qt_ML, Tl_ML and ERA Zinv, and

§ follows the above scheme to produce as outputs Tl(z) and qt(z).  

o Make a second function that:

§ takes the inputs: 

§ Tl_era(z), qt_era(z), qt_ML, Tl_ML and Zinv along with LWP_target.

§ calls the first function to compute Tl(z) and qt(z)

§ computes the LWP of the resulting profile by computing LWC using saturation adjustment at each height, and

§ outputs a (positive) number that tells how well the resulting profile matches LWP_target while preserving the vertical integrals of the ERA Tl and qt profiles.
 
 ![image](https://user-images.githubusercontent.com/28571068/114353655-1ea11a80-9b22-11eb-8096-4d759cf7be45.png)

where Trho is density temeprature:

![image](https://user-images.githubusercontent.com/28571068/114353678-2660bf00-9b22-11eb-8ac2-09bd2a062c7a.png)


  
o Automate the process of determining qt_ML and Tl_ML by using a function like MATLAB's fminsearch that will vary those inputs and choose the values that minimize the output function. 
