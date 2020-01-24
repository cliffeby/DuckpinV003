
### _Introduction_
In [Duckpin Phase I](https://cliffeby.github.io/DuckpinV001/), I described the hardware, software and initial results of a project to illuminate the Lucite numbers on the headboards for Congressional Country Club’s duckpin bowling alleys. [Duckpin Phase II](https://cliffeby.github.io/DuckpinV002/) was an update to the project and focused on tools for analysis, improvement to the streaming framerate, and alternative configurations for detecting the ball count and setter and reset actions.  This Phase III document is an analysis of over 10,000 rolls at all 10 pins.  Data were captured in season Oct-Mar for 2018-19 and 2019-Jan 22, 2020. 

### _Warning_
![image](https://user-images.githubusercontent.com/1431998/73011582-83b80580-3de2-11ea-88a4-85a16b238d87.png)

### _Background_
As described in the previous documents, I use a Raspberry Pi to capture video of any ball that knocks down one or more pins.  Gutter balls are ignored, and this analysis only looks at rolls that start with 10 pins.  For these cases, it can be assumed that bowlers are trying to hit the pocket on either side of the head (#1) pin.  Video is post-processed nightly, to determine the location, speed and angle of the roll.
The limitations of the data are significant.  Some obvious data concerns are:
-	Bowlers are left-handed, right-handed, and two-handed.  The data does not identify.
-	Some bowlers use bumpers and these rolls are not identified.
-	The video quality of a Raspberry Pi depends on resolution.  I used a 1440 x 912-pixel video.  Framerate varied based on Raspberry Pi processor activity.  On average, three images of the ball are captured as it moves to the pins and the moving ball is granular.  Graphics software computes the centroid of the ball which is not always precise.
- Only three seconds of video is captured after a pin is knocked down.  The effect of some pinning pins is not captured.
-	Lane 4 at Congressional CC slopes left.  This is observed in the data and by rolling a ball slowly down the lane.

### _Summary_
Machine learning, AI, and cyber metrics may work in Moneyball, but duckpin bowling has never been about money.  With over 10,000 observations, it appears there are no secrets in the data.  
Data show that ball location is the predominate variable and any method that allows a repeatable roll will achieve results. Forget about the benefits of speed or spin. Focus on a comfortable motion that is consistent.
### _Captured Data_ 
Phase I explains how nightly postprocessing of video data is analyzed and stored in an Azure table. The json data contain xy pairs of the ball locations that produce the endingPinCount. The format of those records is:
~~~ Python
{'PartitionKey': 'Lane 4', 'RowKey': '20180927643118', 'beginingPinCount': 1023, 'endingPinCount': 0, 'x0': '634', 'y0': '829', 'x1': '637', 'y1': '702', >'x2': '641', 'y2': '596', 'x3': '642', 'y3': '510', 'x4': '576', 'y4': '306'}
~~~

The number of xy pairs depends on the speed of the roll.  These data are shaped and establish the following data elements:
-	Location of the ball -x
    -	A duckpin lane is 41” wide.  Ranges for raw x-pixel locations were 60-1150.  Thus, one pixel is a little less than 1/25 on an inch.  The x-location of the second ball image was used – typically about two feet in front of the headpin.  Data were normalized to inches left (-) and right (+) of the lane’s center. 
-	Speed of the ball -v
    -	Calculation of xy pair differences established a relative estimate of speed.  V1s (xy2-xy1) were used for most analysis as faster balls often on had only two xy pairs.  No attempt was made to translate the nominal values to mph or fps.
-	Angle of the ball -theta
    - Calculation of xy pair differences established an estimate of the ball angle as it approached the pins.  Theta1s (xy2-xy1) were used for analysis.
-	Up
    - The ending pin configuration determined the number of pins remaining.  There are 10^2 = 1024 combinations of pins, many of which are impossible to create. 

### _Data Analysis_ 
Analyses are grouped into five categories for all rolls and rolls above average speed.  The later is an attempt to eliminate the effect that bumpers may have on the data:
+	Distribution of all rolls by:
    +	Velocity and X-location
    +	X- location
    +	Pins up
    +	V1
-	Ball location and expected result
-	Effect of speed
-	Effect of approach angle/spin
-	Ball location, deviation and speed for typical ending pin configurations.

#### _Roll Distribution_ #
Figures No. 1 and 2 show that rolls with above average speed (shown by the dashed line) have a slightly better center concentration.  This is likely due to better bowlers tend to throw the ball faster.  Also, noted is the slower average v2 speed.  This is like due to faster rolls often do not capture three xy pairs and v2 of the faster ball was not recorded.

##### Figure No. 1 #
![V1vXScatter](https://user-images.githubusercontent.com/1431998/72767995-f6976580-3bc3-11ea-9813-5a7b9931cf79.png)

##### Figure No. 2 #
![V2vXScatter](https://user-images.githubusercontent.com/1431998/72767997-f6976580-3bc3-11ea-9441-b326ed9fe004.png)


Figures Nos. 3 and 4 are histograms of ball counts by location.  The left-side bias is notable and likely due to the left sloping lane. The predominance of right-handed bowlers could be a factor as well. Figure 3 shows all rolls in blue; Figure 4 is rolls at above average speed.  

##### Figure No. 3 #
![XScatter](https://user-images.githubusercontent.com/1431998/72768003-f72ffc00-3bc3-11ea-9a22-96cbb5ad1932.png)

##### Figure No. 4 #
![XFastScatter](https://user-images.githubusercontent.com/1431998/73036691-571fe000-3e1a-11ea-8aff-bc3b9ab158fd.png) 

Figure No. 5 is the distribution of results for all rolls.  It is the number of pins that remain up after the first roll.  Since gutter ball are not recognized, the maximum number of up pins is nine.  Note that nine pins up is the most frequent roll (mode) and that ten down is the most difficult, but that the relation between 0 and nine is not linear. 
##### Figure No. 5 #
![PisUpHist](https://user-images.githubusercontent.com/1431998/72767987-f5fecf00-3bc3-11ea-9ca1-02b5338e9ff4.png)

Figure No. 6 shows the relationship between v1 and v2.  The data show an unlikely slowdown of rolls at velocities of 75 xps.  This is likely due to parallax as the distance from the camera to the ball is greater on faster rolls.
##### Figure No. 6 #
![V1vsV2Scatter](https://user-images.githubusercontent.com/1431998/72767994-f6976580-3bc3-11ea-8b78-9cd7114289bb.png)
 
#### _Ball location and expected result_ #
Duckpin bowling is unique with its often-unexpected results.  Frequently, a ball that appears to be perfectly located in the pocket leaves two or three pins and sometimes difficult splits.  Also, it’s not unexpected to see a “backdoor” strike where the ball misses the headpin, but pin action delivers a strike.  The following 20 graphs (Figures 7-26) show the distribution of up pins after a roll at X.  The title shows the measurement from the center and the average number of pins up.  These variables are also shown on the x-axis and by the dashed line, respectively. 
##### Figure Nos. 7-26 #
<img src = "https://user-images.githubusercontent.com/1431998/72767966-f4cda200-3bc3-11ea-90e4-f3ffad5da0eb.png" width = "415px" align="left">

<img src = "https://user-images.githubusercontent.com/1431998/72767967-f4cda200-3bc3-11ea-90bb-08535f21c7f5.png" width = "415px" align="right">

<img src = "https://user-images.githubusercontent.com/1431998/72767968-f4cda200-3bc3-11ea-9f12-e448b5aafa61.png" width = "415px" align="left">

<img src = "https://user-images.githubusercontent.com/1431998/72767969-f4cda200-3bc3-11ea-87d9-a7a4f865689c.png" width = "415px" align="right">

<img src = "https://user-images.githubusercontent.com/1431998/72767970-f4cda200-3bc3-11ea-83d1-fb282ab69cff.png" width = "415px" align="left">

<img src = "https://user-images.githubusercontent.com/1431998/72767972-f5663880-3bc3-11ea-9d0e-d115739fe6eb.png" width = "415px" align="right">

<img src = "https://user-images.githubusercontent.com/1431998/72767973-f5663880-3bc3-11ea-95b7-29cdecfa4920.png" width = "415px" align="left">

<img src = "https://user-images.githubusercontent.com/1431998/72767974-f5663880-3bc3-11ea-87d0-3e01a2df39b2.png" width = "415px" align="right">



<img src = "https://user-images.githubusercontent.com/1431998/72767975-f5663880-3bc3-11ea-843e-9b6ec04332bc.png" width = "415px" align="left">


<img src = "https://user-images.githubusercontent.com/1431998/72767976-f5663880-3bc3-11ea-9355-f289899a54e5.png" width = "415px" align="right">

<img src = "https://user-images.githubusercontent.com/1431998/72767977-f5663880-3bc3-11ea-9242-ab3efeeea547.png" width = "415px" align="left">

<img src = "https://user-images.githubusercontent.com/1431998/72767978-f5663880-3bc3-11ea-861d-98f8df4acf3b.png" width = "415px" align="right">

<img src = "https://user-images.githubusercontent.com/1431998/72767979-f5663880-3bc3-11ea-8ffc-0db4f6ac7c12.png" width = "415x" align="left">

<img src = "https://user-images.githubusercontent.com/1431998/72767980-f5663880-3bc3-11ea-885a-f261933c6b92.png" width = "415px" align="right">

<img src = "https://user-images.githubusercontent.com/1431998/72767981-f5fecf00-3bc3-11ea-91a1-228a3777cf20.png" width = "415px" align="left">

<img src = "https://user-images.githubusercontent.com/1431998/72767982-f5fecf00-3bc3-11ea-9537-3c6b15637130.png" width = "415px" align="right">

<img src = "https://user-images.githubusercontent.com/1431998/72767982-f5fecf00-3bc3-11ea-9537-3c6b15637130.png" width = "415px" align="left">

<img src = "https://user-images.githubusercontent.com/1431998/72767985-f5fecf00-3bc3-11ea-8eef-fe60ed42622a.png" width = "415px" align="right">

#### _Effect of speed_ #
Can a duckpin roll be too fast?  While the higher speed departs more energy to the pins it hits, can the ball be traveling too fast to impart maximum results.  Do pins fall into the well or spin on the metal table more often at higher speeds? 
  
Figures 27 and 28 show the effect of speed on the number of pins remaining. The average speed and standard deviation bars are plotted for each ending state.  For all balls, Figure 27 shows that speed may have some benefit for strikes through four pins up.  When above average speed is the control variable, The already small effect is minimized.  A difference in speed of 161 to 148 - nine percent or about one percent per pin.
##### Figure No. 27 #
![speed](https://user-images.githubusercontent.com/1431998/72767988-f5fecf00-3bc3-11ea-8dbb-09dbb61cd66c.png)

##### Figure No. 28 #
![SpeedFast](https://user-images.githubusercontent.com/1431998/72767990-f6976580-3bc3-11ea-814e-bb2b20aee167.png)

A second approach is shown in Figure 29.  It shows the speed and location of all rolls that are strikes.  It appears that there is a benefit to faster speed but that some well off-center slow balls have a better strike chance. 
##### Figure No. 29 # 
![StrikeSpeed1](https://user-images.githubusercontent.com/1431998/73009866-49009e00-3ddf-11ea-9123-18295047e02d.png)

#### Effect of approach angle/spin #
Hook bowling is the technique used by almost every professional 10-pin bowler. Using the hook allows you much more control over where the ball strikes the pins and where the pins are hit to as a result. In 10-pins the ball is big enough to contact the #1-#3 or #1-#2 pins simultaneously if hooked.  In duckpins, simultaneous contact is impossible.  It is also believed that the angle offers a better chance of direct contact with the #5 pin.  Finally, the spin is believed to impart rotational energy on the pins.  Some even believe that a backwards spinning ball on a well-oiled lane is best. 

##### Figure No. 29 # 
<img src= "https://user-images.githubusercontent.com/1431998/72758770-baeba400-3ba1-11ea-8d79-388a09c3b466.png" width = "430px" align = "center">

By comparing xy pairs, I was able to detect the angle of attack.  The Raspberry Pi camera is not capable of the hi-res recording needed to determine spin.  Also, left- vs right-handed throws are not distinguished so the best measure of angle is the absolute value of the calculated angle.
Figure 30 shows no benefit of angle/spin for above average speed rolls.  Data for all rolls as well as left- and right-angled approaches were similar.

##### Figure No. 30 # 
![AngleFastAbs](https://user-images.githubusercontent.com/1431998/72768007-f72ffc00-3bc3-11ea-9bc5-e7d857860d71.png)

 
The distribution of ball angles is shown in Figure 31.  It likely reflects the predominance of right-handed bowlers and the left leaning Lane 4.
##### Figure No. 31 # 
![AngleHist](https://user-images.githubusercontent.com/1431998/72768008-f72ffc00-3bc3-11ea-8c5e-478600f52349.png)

#### Ball location, deviation and speed for typical ending pin configurations #
The remaining figures are not analyzed but show the most common pin configurations in two Appendices.
Each Figure contains the ball track and location where its size is a standard deviation, a table of stats, and a scatter diagram of all throws.   The population is the total number of rolls for that configuration and is presented in rank order. 
Appendix 1 contains the most common outcomes and Appendix 2 shows some least common outcomes.  I refer to these as Bad Beats and Splits.  While there is only a one in 10,000 chance of these outcomes, it seems that I've had thme all.  Note that often ball speed is slow.

### _Appendix 1 - Most Common Outcomes for 10 Pins_ #
![Figure_1](https://user-images.githubusercontent.com/1431998/73009876-49993480-3ddf-11ea-8ff2-029437100fce.png)

![Figure_2](https://user-images.githubusercontent.com/1431998/73009879-49993480-3ddf-11ea-9a55-814a299258c6.png)

![Figure_3](https://user-images.githubusercontent.com/1431998/73009881-4a31cb00-3ddf-11ea-97c5-f0facbdf5a1f.png)

![Figure_4](https://user-images.githubusercontent.com/1431998/73009884-4a31cb00-3ddf-11ea-850a-d236d3fbf368.png)

![Figure_5](https://user-images.githubusercontent.com/1431998/73009886-4a31cb00-3ddf-11ea-9922-7799c0f86bc1.png)

![Figure_6](https://user-images.githubusercontent.com/1431998/73009888-4a31cb00-3ddf-11ea-964d-6175cf492655.png)

![Figure_7](https://user-images.githubusercontent.com/1431998/73009890-4a31cb00-3ddf-11ea-9aff-0eed963c490a.png)

![Figure_8](https://user-images.githubusercontent.com/1431998/73009893-4a31cb00-3ddf-11ea-9ae3-0e651729df15.png)

![Figure_9](https://user-images.githubusercontent.com/1431998/73009895-4aca6180-3ddf-11ea-91ec-240389f0bf17.png)

![Figure_10](https://user-images.githubusercontent.com/1431998/73009836-47cf7100-3ddf-11ea-934a-a7b62482d273.png)

![Figure_11](https://user-images.githubusercontent.com/1431998/73009838-47cf7100-3ddf-11ea-8215-f8e1bea3ee78.png)



### _Appendix 2 - Bad Beats for 10 Pins_ #
![Figure_1B](https://user-images.githubusercontent.com/1431998/73009878-49993480-3ddf-11ea-8526-5246b917aa00.png)
![Figure_2B](https://user-images.githubusercontent.com/1431998/73009880-4a31cb00-3ddf-11ea-85eb-35c43cc84cf3.png)
![Figure_3B](https://user-images.githubusercontent.com/1431998/73009882-4a31cb00-3ddf-11ea-959c-4f62da2ba899.png)
![Figure_4B](https://user-images.githubusercontent.com/1431998/73009885-4a31cb00-3ddf-11ea-8756-02d2c1238c53.png)
![Figure_5B](https://user-images.githubusercontent.com/1431998/73009887-4a31cb00-3ddf-11ea-875b-7db40952fdf7.png)
![Figure_6B](https://user-images.githubusercontent.com/1431998/73009889-4a31cb00-3ddf-11ea-80e5-8ffc5b0ec257.png)
![Figure_7B](https://user-images.githubusercontent.com/1431998/73009892-4a31cb00-3ddf-11ea-9aae-5c676b8e8af2.png)
![Figure_8B](https://user-images.githubusercontent.com/1431998/73009894-4a31cb00-3ddf-11ea-8f34-fd80f41a0f41.png)
![Figure_9B](https://user-images.githubusercontent.com/1431998/73009835-47cf7100-3ddf-11ea-92fb-9406a45b1a12.png)
![Figure_10B](https://user-images.githubusercontent.com/1431998/73009837-47cf7100-3ddf-11ea-85be-d8d7ec0c6033.png)
![Figure_11B](https://user-images.githubusercontent.com/1431998/73009839-47cf7100-3ddf-11ea-8787-b9ba0532a1ce.png)



test

![v1tov2](https://user-images.githubusercontent.com/1431998/72767993-f6976580-3bc3-11ea-9e22-fc109d2cf9ee.png)


![PerfectStrike](https://user-images.githubusercontent.com/1431998/72768031-f8f9bf80-3bc3-11ea-9c28-b5cf907b08fd.png)
![xHistGT117](https://user-images.githubusercontent.com/1431998/72768001-f72ffc00-3bc3-11ea-96d4-2e0bd2d01297.png)

![AllBalls](https://user-images.githubusercontent.com/1431998/72768004-f72ffc00-3bc3-11ea-99e7-40565692e45e.png)




