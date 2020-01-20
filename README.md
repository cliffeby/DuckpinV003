
### _Introduction_
In [Duckpin Phase I](https://cliffeby.github.io/DuckpinV001/), I described the hardware, software and initial results of a project to illuminate the Lucite numbers on the headboards for Congressional Country Club’s duckpin bowling alleys. [Duckpin Phase II](https://cliffeby.github.io/DuckpinV002/) was an update to the project and focused on tools for analysis, improvement to the streaming framerate, and alternative configurations for detecting the ball count and setter and reset actions.  This Phase III document is an analysis of over 8,000 rolls at all 10 pins.  Data were captured in season Oct-Mar for 2018-19 and 2019-Jan 20, 2020. 

### _Background_
As described in the previous documents, I use a Raspberry Pi to capture video of any ball that knocks down one or more pins.  Gutter balls are ignored, and this analysis only looks at rolls that start with 10 pins.  For these cases, it can be assumed that bowlers are trying to hit the pocket on either side of the head (#1) pin.  Video is post-processed nightly, to determine the location, speed and angle of the roll.
The limitations of the data are significant.  Some obvious data concerns are:
-	Bowlers are left-handed, right-handed, and two-handed.  The data does not identify.
-	Some bowlers use bumpers and these rolls are not identified.
-	The video quality of a Raspberry Pi depends on resolution.  I used a 1440 x 912-pixel video.  Framerate varied based on Raspberry Pi processor activity.  On average, three images of the ball are captured as it moves to the pins and the moving ball is granular.  Graphics software computes the centroid of the ball which is not always precise.
-	Lane 4 at Congressional CC slopes left.  This is observed in the data and by rolling a ball slowly down the lane.

### _Summary_
Machine learning, AI, and cyber metrics may work in Moneyball, but duckpin bowling has never been about money.  With over 8000 observations, it appears there are no secrets in the data.  
Data show that ball location is the predominate variable and any method that allows a repeatable roll will achieve results. Forget about the benefits of speed or spin. Focus on a comfortable motion that is consistent.
Captured Data 
Phase I explains how nightly postprocessing of video data is analyzed and stored in an Azure table. The json data contain xy pairs of the ball locations that produce the endingPinCount. The format of those records is:
{'PartitionKey': 'Lane 4', 'RowKey': '20180927643118', 'beginingPinCount': 1023, 'endingPinCount': 0, 'x0': '634', 'y0': '829', 'x1': '637', 'y1': '702', >'x2': '641', 'y2': '596', 'x3': '642', 'y3': '510', 'x4': '576', 'y4': '306'}

The number of xy pairs depends on the speed of the roll.  These data are shaped and establish the following data elements:
-	Location of the ball -x
o	A duckpin lane is 41” wide.  Ranges for raw x-pixel locations were 60-1150.  Thus, one pixel is a little less than 1/25 on an inch.  The x-location of the second ball image was used – typically about two feet in front of the headpin.  Data were normalized to inches left (-) and right (+) of the lane’s center. 
-	Speed of the ball -v
o	Calculation of xy pair differences established a relative estimate of speed.  V1s (xy2-xy1) were used for most analysis as faster balls often on had two xy pairs.  No attempt was made to translate the nominal values to mph or fps.
-	Angle of the ball -theta
o	 Calculation of xy pair differences established an estimate of the ball angle as it approached the pins.  Theta1s (xy2-xy1) were used for analysis.
-	Up
o	The ending pin configuration determined the number of pins remaining.  There are 10^2 = 1024 combinations of pins, many of which are impossible to create. 

### _Data Analysis_ 
Analyses are grouped into five categories for all rolls and rolls above average speed.  The later is an attempt to eliminate the effect that bumpers may have on the data:
-	Distribution of all rolls by:
o	Velocity and X-location
o	X- location
o	Pins up
o	V1
-	Ball location and expected result
-	Effect of speed
-	Effect of approach angle/spin
-	Ball location, deviation and speed for typical ending pin configurations.
#### Roll Distribution_
Figure 1 and 2 show that rolls with above average speed (shown by the dashed line) have a slightly better center concentration.  This is likely due to better bowlers tend to throw the ball faster.  Also, noted is the slower average v2 speed.  This is like due to faster rolls often do not capture three xy pairs and v2 was unknown.
Figure 3 and 4 are histograms of ball counts by location.  The left-side bias is notable and likely due to the left sloping lane. The predominance of right-handed bowlers could be a factor as well. Figure 3 shows all rolls in blue; Figure 4 is rolls at above average speed.  
 
Figure 5 is the distribution of results for all rolls.  It is the number of pins that remain up after the first roll.  Since gutter ball are not recognized, the maximum number of up pins is nine.  Note that seven pins up is the most frequent roll (mode) and that it is just as hard to knock down only four pins as it is to get all ten. 

Figure 6 shows the relationship between v1 and v2.  The data show an unlikely slowdown of rolls at velocities of 75 xps.  This is likely due to the distance from the camera to the ball on faster rolls. 

 

Ball location and expected result
Duckpin bowling is unique with its often-unexpected results.  Often a ball that appears to be perfectly located in the pocket leaves two or three pins and sometimes difficult splits.  Also, it’s not unexpected to see a “backdoor” strike where the ball misses the headpin, but pin action delivers a strike.  The following 20 graphs (Figures 7-26) show the distribution of up pins after a roll at X.  The title shows the measurement from the center and the average number of pins up.  These variables are also shown on the x-axis and by the dashed line, respectively. 
  

Effect of speed
Can a duckpin roll be too fast?  While the higher speed departs more energy to the pins it hits, can the ball be traveling too fast to impart maximum results.  Do pins fall into the well or spin on the metal table more often at higher speeds? 
  
Figures 27 and 28 show the effect of speed on the number of pins remaining. The average speed and standard deviation bars are plotted for each ending state.  For all balls, Figure 27 shows that speed may have some benefit for strikes through four pins up.  When above average speed is the control variable, The already small effect is minimized.  A difference in speed of 161 to 148 - nine percent or about one percent per pin.

A second approach is shown in Figure 29.  It shows the speed and location of all rolls that are strikes.  It appears that there is a benefit to faster speed but that some well off-center slow balls have a better strike chance. 
Effect of approach angle/spin
Hook bowling is the technique used by almost every professional 10-pin bowler. Using the hook allows you much more control over where the ball strikes the pins and where the pins are hit to as a result. In 10-pins the ball is big enough to contact the #1-#3 or #1-#2 pins simultaneously if hooked.  In duckpins, simultaneous contact is impossible.  It is also believed that the angle offers a better chance of direct contact with the #5 pin.  Finally, the spin is believed to impart rotational energy on the pins.  Some even believe that a backwards spinning ball on a well-oiled lane is best.  
By comparing xy pairs, I was able to detect the angle of attack.  The Raspberry Pi camera is not capable of the hi-res recording needed to determine spin.  Left- vs right-handed throws are not distinguished so the best measure of angle is the absolute value of the calculated angle.
Figure 30 shows no benefit of angle/spin for above average speed rolls.  Data for all rolls as well as left- and right-angled approaches were similar.

 
The distribution of ball angles is shown in Figure 31.  It likely reflects the predominance of right-handed bowlers and the left leaning Lane 4.
 
Ball location, deviation and speed for typical ending pin configurations.

The remaining Figures are not analyzed but show the most common pin configurations.
Each Figure contains the ball location where its size is a standard deviation, a table of stats, and a scatter diagram of all throws.   The population is the total number of rolls for that configuration and is presented in that order. 

