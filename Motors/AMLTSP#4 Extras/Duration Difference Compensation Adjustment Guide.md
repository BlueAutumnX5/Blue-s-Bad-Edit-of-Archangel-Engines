This is a short duration difference compensation adjustment guide

Duration difference compensation, also known as "STS0050D" line, is responsible for altering the intake and exhaust to take advantage of full intake and exhaust duration as opposed to just 0.050 duration.
As a reference, default template engines without any changes require a value of 40.0.


Before you start the adjustment process make sure to input every parameter you can since there might be a need to readjust it if you change just about anything geometry-related.
I do not have a full list of parameters that affect it, but what I know is that bore, stroke, connecting rod length, and valve lift definitely affect the required compensation.

Terms:
STS - Seat to seat duration
Adv. - Advertised as in "Advertised duration:
Timing marks - IVO, IVC, EVO, EVC


Normal Procedure:

Let's first deal with "Why?" and "How?".
Since STS(Adv.) duration is higher than 0.050 duration, setting your IVO to 0.0 does not mean that intake valve starts opening at TDC. It only means that it is open by 0.050 inches at TDC. Same goes for every other valve timing mark.
The effects of it can be seen by looking at "Valve Lift" graph which shows some valve overlap (img1.png) even with valve timings set to 0.0(img1.1).
By finding out how much exactly earlier/later the valves open/close we can tell the required compensation value.
The easiest way to find it out is by reducing both intake and exhaust valve durations equally until the overlap is gone.

It is always wise to start slow. I reduced valve durations by a total of 20 degrees (-10.0 on IVO, IVC, EVO, EVC (img2.1)). Time to see what happens.
Well, the valve overlap on the graph is reduced but still present (img2.png). This is not what we are looking for.

Lower duration time! The valve durations are now reduced by a total of 60 degrees(-30.0 on IVO, IVC, EVO, EVC (img3.1)). Let's see the result.
Graph reads complete absence of overlap (img3.png), but unfortunately the durations are so low that there is now an area where valves do nothing at all. This is way more than what we need.

Let's make the durations slightly longer again. I have used a value of -20.0 on IVO, IVC, EVO, and EVC (img4.1), meaning that I reduced durations by a total of 40 degrees.
Taking a look at the graph, I can immediately see that there is no overlap, but this time intake valve starts opening as soon as exhaust valve is fully closed (img4.png). This is perfect!
As a side note, intake and exhaust valve both close right at BDC now.

What this all means is that to achieve 180/180 STS(Adv.) intake and exhaust duration, we had to make the duration at 0.050 shorter by a total of 40 degrees. This is the "STS0050D" value we were looking for.
You can write it down and revert all the changes you have made to IVO, IVC, EVO, and EVC.

Minor clarifications:
The difference is measured on a single lobe, not both of them. It can be measured as |IVO| + |EVC| or |IVO| + |IVC| or |EVO| + |EVC| if you subrtracted duration from all of the marks evenly.
When putting in the value you do not need a plus or a minus sign.


What you do next varies depending on what camshaft you want to make. Here are the most common scenarios and your respective courses of action:

A: You are making a custom camshaft but you prefer to measure duration at 0.050.
In this case, you set IVO, IVC, EVO, EVC the way you normally would, but still plug in the "STS0050D" value.


B: You are making a custom camshaft and you prefer to measure STS(Adv.) duration.
It is still simple. Ignore that IVO, IVC, EVO, EVC values use 0.050 measurement and input your STS(Adv.) values instead. Now substract STS0050D value divided by 2 from all your marks.
Example:
When measured at seat, your intake valve opens 20BTDC and closes 40ABDC, while exhaust valve opens 50BBDC and closes 10ATDC. 
You input those values and get IVO(20.0), IVC(40.0), EVO(50.0), EVC(10.0)
You already know your STS0050D value, which is 40.0 for the sake of easy example. You divide 40.0 by 2 since cam lobes are symmetrical (Hopefully they are in your case) and get 20.0. Now you substract 20.0 from your timing marks and get following values:
IVO(0.0), IVC(20.0). EVO(30.0), EVC(-10.0)
You now have cams specified based on STS(Adv.) duration that are correctly converted to 0.050 used by camshaft builder formulas.


C: You have a cam spec with only 0.050 durations specified.
Refer to scenario A.


D: You have a cam spec with only Adv. duration specified. 
Refer to scenario B.


E: You have a cam spec with both 0.050 and Adv. durations specified.
There are 3 ways to approach this scenario. The laziest option is substracting your specified 0.050 duration from Adv. duration and using the result as "STS0050D" value.
Better option is taking only the Adv. Duration and referring to Scenario B.
Another option is using the 0.050 values and referring to Scenario C.