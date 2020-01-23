
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
- Only three seconds of video is captured after a pin is knocked down.  the effect of some pinning pins is not captured.
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
Figure 1 and 2 show that rolls with above average speed (shown by the dashed line) have a slightly better center concentration.  This is likely due to better bowlers tend to throw the ball faster.  Also, noted is the slower average v2 speed.  This is like due to faster rolls often do not capture three xy pairs and v2 was unknown.

##### Figure No. 1 #
![V1vXScatter](https://user-images.githubusercontent.com/1431998/72767995-f6976580-3bc3-11ea-9813-5a7b9931cf79.png)

##### Figure No. 2 #
![V2vXScatter](https://user-images.githubusercontent.com/1431998/72767997-f6976580-3bc3-11ea-9441-b326ed9fe004.png)


Figure 3 and 4 are histograms of ball counts by location.  The left-side bias is notable and likely due to the left sloping lane. The predominance of right-handed bowlers could be a factor as well. Figure 3 shows all rolls in blue; Figure 4 is rolls at above average speed.  

##### Figure No. 3 #
![XScatter](https://user-images.githubusercontent.com/1431998/72768003-f72ffc00-3bc3-11ea-9a22-96cbb5ad1932.png)

##### Figure No. 4 #
![XFastScatter](https://user-images.githubusercontent.com/1431998/72768000-f72ffc00-3bc3-11ea-8f95-c9e9881e3550.png) 

Figure 5 is the distribution of results for all rolls.  It is the number of pins that remain up after the first roll.  Since gutter ball are not recognized, the maximum number of up pins is nine.  Note that seven pins up is the most frequent roll (mode) and that it is just as hard to knock down only four pins as it is to get all ten. 
##### Figure No. 5 #
![PisUpHist](https://user-images.githubusercontent.com/1431998/72767987-f5fecf00-3bc3-11ea-9ca1-02b5338e9ff4.png)

Figure 6 shows the relationship between v1 and v2.  The data show an unlikely slowdown of rolls at velocities of 75 xps.  This is likely due to the distance from the camera to the ball on faster rolls.
##### Figure No. 6 #
![V1vsV2Scatter](https://user-images.githubusercontent.com/1431998/72767994-f6976580-3bc3-11ea-8b78-9cd7114289bb.png)
 
#### _Ball location and expected result_ #
Duckpin bowling is unique with its often-unexpected results.  Frequently, a ball that appears to be perfectly located in the pocket leaves two or three pins and sometimes difficult splits.  Also, it’s not unexpected to see a “backdoor” strike where the ball misses the headpin, but pin action delivers a strike.  The following 20 graphs (Figures 7-26) show the distribution of up pins after a roll at X.  The title shows the measurement from the center and the average number of pins up.  These variables are also shown on the x-axis and by the dashed line, respectively. 
##### Figure Nos. 7-26 #
<img src = "https://user-images.githubusercontent.com/1431998/72767966-f4cda200-3bc3-11ea-90e4-f3ffad5da0eb.png" width = "400px" align="left">

<img src = "https://user-images.githubusercontent.com/1431998/72767967-f4cda200-3bc3-11ea-90bb-08535f21c7f5.png" width = "400px" align="right">

![PinUpatXHist3](https://user-images.githubusercontent.com/1431998/72767968-f4cda200-3bc3-11ea-9f12-e448b5aafa61.png)
![PinUpatXHist4](https://user-images.githubusercontent.com/1431998/72767969-f4cda200-3bc3-11ea-87d9-a7a4f865689c.png)
![PinUpatXHist5](https://user-images.githubusercontent.com/1431998/72767970-f4cda200-3bc3-11ea-83d1-fb282ab69cff.png)
![PinUpatXHist6](https://user-images.githubusercontent.com/1431998/72767972-f5663880-3bc3-11ea-9d0e-d115739fe6eb.png)
![PinUpatXHist7](https://user-images.githubusercontent.com/1431998/72767973-f5663880-3bc3-11ea-95b7-29cdecfa4920.png)
![PinUpatXHist8](https://user-images.githubusercontent.com/1431998/72767974-f5663880-3bc3-11ea-87d0-3e01a2df39b2.png)
![PinUpatXHist9](https://user-images.githubusercontent.com/1431998/72767975-f5663880-3bc3-11ea-843e-9b6ec04332bc.png)
![PinUpatXHist10](https://user-images.githubusercontent.com/1431998/72767976-f5663880-3bc3-11ea-9355-f289899a54e5.png)
![PinUpatXHist11](https://user-images.githubusercontent.com/1431998/72767977-f5663880-3bc3-11ea-9242-ab3efeeea547.png)
![PinUpatXHist12](https://user-images.githubusercontent.com/1431998/72767978-f5663880-3bc3-11ea-861d-98f8df4acf3b.png)
![PinUpatXHist13](https://user-images.githubusercontent.com/1431998/72767979-f5663880-3bc3-11ea-8ffc-0db4f6ac7c12.png)
![PinUpatXHist14](https://user-images.githubusercontent.com/1431998/72767980-f5663880-3bc3-11ea-885a-f261933c6b92.png)
![PinUpatXHist15](https://user-images.githubusercontent.com/1431998/72767981-f5fecf00-3bc3-11ea-91a1-228a3777cf20.png)
![PinUpatXHist16](https://user-images.githubusercontent.com/1431998/72767982-f5fecf00-3bc3-11ea-9537-3c6b15637130.png)
![PinUpatXHist17](https://user-images.githubusercontent.com/1431998/72767983-f5fecf00-3bc3-11ea-8e38-7e9cf16febb9.png)
![PinUpatXHist18](https://user-images.githubusercontent.com/1431998/72767984-f5fecf00-3bc3-11ea-8458-8cb4efb0fad5.png)
![PinUpatXHist19](https://user-images.githubusercontent.com/1431998/72767985-f5fecf00-3bc3-11ea-8eef-fe60ed42622a.png)  

#### _Effect of speed_ #
Can a duckpin roll be too fast?  While the higher speed departs more energy to the pins it hits, can the ball be traveling too fast to impart maximum results.  Do pins fall into the well or spin on the metal table more often at higher speeds? 
  
Figures 27 and 28 show the effect of speed on the number of pins remaining. The average speed and standard deviation bars are plotted for each ending state.  For all balls, Figure 27 shows that speed may have some benefit for strikes through four pins up.  When above average speed is the control variable, The already small effect is minimized.  A difference in speed of 161 to 148 - nine percent or about one percent per pin.

A second approach is shown in Figure 29.  It shows the speed and location of all rolls that are strikes.  It appears that there is a benefit to faster speed but that some well off-center slow balls have a better strike chance. 
Effect of approach angle/spin
Hook bowling is the technique used by almost every professional 10-pin bowler. Using the hook allows you much more control over where the ball strikes the pins and where the pins are hit to as a result. In 10-pins the ball is big enough to contact the #1-#3 or #1-#2 pins simultaneously if hooked.  In duckpins, simultaneous contact is impossible.  It is also believed that the angle offers a better chance of direct contact with the #5 pin.  Finally, the spin is believed to impart rotational energy on the pins.  Some even believe that a backwards spinning ball on a well-oiled lane is best. 
<img src= "https://user-images.githubusercontent.com/1431998/72758770-baeba400-3ba1-11ea-8d79-388a09c3b466.png" width = "430px" align = "center">
By comparing xy pairs, I was able to detect the angle of attack.  The Raspberry Pi camera is not capable of the hi-res recording needed to determine spin.  Left- vs right-handed throws are not distinguished so the best measure of angle is the absolute value of the calculated angle.
Figure 30 shows no benefit of angle/spin for above average speed rolls.  Data for all rolls as well as left- and right-angled approaches were similar.

 
The distribution of ball angles is shown in Figure 31.  It likely reflects the predominance of right-handed bowlers and the left leaning Lane 4.
 
Ball location, deviation and speed for typical ending pin configurations.

The remaining Figures are not analyzed but show the most common pin configurations.
Each Figure contains the ball location where its size is a standard deviation, a table of stats, and a scatter diagram of all throws.   The population is the total number of rolls for that configuration and is presented in that order. 


![PinUpHistforBallatX10](https://user-images.githubusercontent.com/1431998/72767986-f5fecf00-3bc3-11ea-8a28-ae9933b54662.png)

![speed](https://user-images.githubusercontent.com/1431998/72767988-f5fecf00-3bc3-11ea-8dbb-09dbb61cd66c.png)
![Speed1](https://user-images.githubusercontent.com/1431998/72767989-f6976580-3bc3-11ea-976b-315097a6445a.png)
![SpeedFast](https://user-images.githubusercontent.com/1431998/72767990-f6976580-3bc3-11ea-814e-bb2b20aee167.png)
![StrikeSpeed](https://user-images.githubusercontent.com/1431998/72767991-f6976580-3bc3-11ea-8ac6-7b065b12289b.png)
![ThetaScatter](https://user-images.githubusercontent.com/1431998/72767992-f6976580-3bc3-11ea-9380-f34b1ffd9026.png)
![v1tov2](https://user-images.githubusercontent.com/1431998/72767993-f6976580-3bc3-11ea-9e22-fc109d2cf9ee.png)



![xHistGT117](https://user-images.githubusercontent.com/1431998/72768001-f72ffc00-3bc3-11ea-96d4-2e0bd2d01297.png)

![AllBalls](https://user-images.githubusercontent.com/1431998/72768004-f72ffc00-3bc3-11ea-99e7-40565692e45e.png)
![Angle](https://user-images.githubusercontent.com/1431998/72768005-f72ffc00-3bc3-11ea-9d6a-dc84eb548fb3.png)
![AngleFast](https://user-images.githubusercontent.com/1431998/72768006-f72ffc00-3bc3-11ea-9d2b-a7608bbe7884.png)
![AngleFastAbs](https://user-images.githubusercontent.com/1431998/72768007-f72ffc00-3bc3-11ea-9bc5-e7d857860d71.png)
![AngleHist](https://user-images.githubusercontent.com/1431998/72768008-f72ffc00-3bc3-11ea-8c5e-478600f52349.png)
![Figure_1](https://user-images.githubusercontent.com/1431998/72768009-f7c89280-3bc3-11ea-9171-983251d70293.png)
![Figure_1a](https://user-images.githubusercontent.com/1431998/72768010-f7c89280-3bc3-11ea-8641-2d9a54cfd141.png)
![Figure_1b](https://user-images.githubusercontent.com/1431998/72768011-f7c89280-3bc3-11ea-828a-c5e5c1e84d84.png)
![Figure_2](https://user-images.githubusercontent.com/1431998/72768012-f7c89280-3bc3-11ea-8e3c-8e93470493c3.png)
![Figure_2a](https://user-images.githubusercontent.com/1431998/72768013-f7c89280-3bc3-11ea-86de-53e98f4a7263.png)
![Figure_3](https://user-images.githubusercontent.com/1431998/72768014-f7c89280-3bc3-11ea-8bf6-dadeac078ef3.png)
![Figure_3a](https://user-images.githubusercontent.com/1431998/72768015-f7c89280-3bc3-11ea-9463-fc90b611f88e.png)
![Figure_4](https://user-images.githubusercontent.com/1431998/72768016-f7c89280-3bc3-11ea-8170-bd28ca8aa221.png)
![Figure_4a](https://user-images.githubusercontent.com/1431998/72768017-f7c89280-3bc3-11ea-8300-4fa656d5075b.png)
![Figure_5](https://user-images.githubusercontent.com/1431998/72768018-f8612900-3bc3-11ea-8aa8-6273756485c1.png)
![Figure_5a](https://user-images.githubusercontent.com/1431998/72768019-f8612900-3bc3-11ea-88d6-f6cc290cb57b.png)
![Figure_6](https://user-images.githubusercontent.com/1431998/72768020-f8612900-3bc3-11ea-82f0-12c7d36255c1.png)
![Figure_6a](https://user-images.githubusercontent.com/1431998/72768022-f8612900-3bc3-11ea-8e7d-3ae76809483f.png)
![Figure_7](https://user-images.githubusercontent.com/1431998/72768023-f8612900-3bc3-11ea-9c39-a4d2fab2870c.png)
![Figure_7a](https://user-images.githubusercontent.com/1431998/72768024-f8612900-3bc3-11ea-9ae2-b3d92b8e7ba9.png)
![Figure_8](https://user-images.githubusercontent.com/1431998/72768025-f8612900-3bc3-11ea-8c3f-c6287a80d955.png)
![Figure_8a](https://user-images.githubusercontent.com/1431998/72768026-f8612900-3bc3-11ea-9de7-afdcb5286da1.png)
![Figure_9](https://user-images.githubusercontent.com/1431998/72768027-f8612900-3bc3-11ea-86f9-199f40dcce55.png)
![Figure_9a](https://user-images.githubusercontent.com/1431998/72768028-f8612900-3bc3-11ea-8158-7acf8f4f653a.png)
![Figure_10](https://user-images.githubusercontent.com/1431998/72768029-f8612900-3bc3-11ea-818e-d576d84300d1.png)
![Figure_10a](https://user-images.githubusercontent.com/1431998/72768030-f8f9bf80-3bc3-11ea-9828-b13abb399b5c.png)
![PerfectStrike](https://user-images.githubusercontent.com/1431998/72768031-f8f9bf80-3bc3-11ea-9c28-b5cf907b08fd.png)
![PinFigure_1](https://user-images.githubusercontent.com/1431998/72768032-f8f9bf80-3bc3-11ea-9e05-4aa4ae093f10.png)
![PinFigure_2](https://user-images.githubusercontent.com/1431998/72768033-f8f9bf80-3bc3-11ea-9bc8-4cda3252ccc1.png)
![PinFigure_3](https://user-images.githubusercontent.com/1431998/72768034-f8f9bf80-3bc3-11ea-9770-dd06c6012434.png)
![PinFigure_4](https://user-images.githubusercontent.com/1431998/72768035-f8f9bf80-3bc3-11ea-96d3-7752c06a4a27.png)
![PinFigure_5](https://user-images.githubusercontent.com/1431998/72768036-f8f9bf80-3bc3-11ea-8262-d00dc6af9e09.png)
![PinFigure_6](https://user-images.githubusercontent.com/1431998/72768037-f9925600-3bc3-11ea-826b-380cc6c7fea7.png)
![PinFigure_7](https://user-images.githubusercontent.com/1431998/72768038-f9925600-3bc3-11ea-8153-facb58cf0a6a.png)
![PinFigure_8](https://user-images.githubusercontent.com/1431998/72768039-f9925600-3bc3-11ea-9716-fdfb7f09c20a.png)
![PinFigure_9](https://user-images.githubusercontent.com/1431998/72768040-f9925600-3bc3-11ea-8e71-75e6865caea0.png)
![PinFigure_9a](https://user-images.githubusercontent.com/1431998/72768041-f9925600-3bc3-11ea-9d63-489b0ca8dbbd.png)
![PinFigure_10](https://user-images.githubusercontent.com/1431998/72768042-f9925600-3bc3-11ea-978a-a8cf68dd7d58.png)
![PinsUPHist](https://user-images.githubusercontent.com/1431998/72768043-f9925600-3bc3-11ea-8463-cd2621c97560.png)
![PinUpatXHist0](https://user-images.githubusercontent.com/1431998/72768044-f9925600-3bc3-11ea-91c6-55b22f29917a.png)
