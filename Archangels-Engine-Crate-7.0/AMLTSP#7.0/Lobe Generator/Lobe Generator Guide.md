# Oror's Lobe Generator Guide



## Installation
1) Install Python interpreter, this could be found on python.org for any platform.
2) Run PowerShell or any other terminal and check that py is installed:  py --version
3) Install script dependencies: py -m pip install scipy numpy matplotlib parse argparse comment_parser regex


## Usage

1) Set your valve timing events and generation parameters in the engine file.mr to your desired values, everything except resolution can be changed in the GUI, more info in "Generator Parameters";
2) In the folder with your script file, press Shift+RMB and select "Open Powershell window here" in the context menu; 
3) Run the following command: py lobesAMMC.py -ar , If you are using non-AMMC engine, use this command: py lobes.py ;
4) Select the file you want to edit when prompted;
5) Adjust the lobe generation parameters as you desire using the GUI;
6) Click "Save" and exit the lobe generator. All the changes are applied to your engine file automatically.


## Generator Parameters 

Here is what the config looks like in the code, but with explanations added.
Keep in mind that everything except resolution can be adjusted in the Lobe Generator GUI when you run it.

/*
{
   "cam_cfg":{
      "es_version":"0.1.14a",
      "resolution":100,                 Amount of samples for your lobe function. Higher is better, but takes longer to load.
      "intake_volume":1.0,              Overal intake lobe volume. Higher - fatter lobe and vice versa
      "exhaust_volume":1.0,             Overal exhaust lobe volume. Higher - fatter lobe and vice versa
      "intake_at_lift":0.0,             Lift at which intake duration is specified. If set to 1.27(0.050), your IV_O, IV_C will dictate at what point the valve is open/closed by/to 1.27(0.050).
      "exhaust_at_lift":0.0,            Lift at which exhaust duration is specified. If set to 1.27(0.050), your EV_O, EV_C will dictate at what point the valve is open/closed by/to 1.27(0.050).
      "intake_trim":1.0,                Trims the intake lobe forming a flat surface instead of its tip. 1.0 Is no trimming, 0.0 Breaks the lobe. Anything in between works, check it out.
      "exhaust_trim":1.0,               Trims the exhaust lobe forming a flat surface instead of its tip. 1.0 Is no trimming, 0.0 Breaks the lobe. Anything in between works, check it out.
      "intake_sigma":1.0,               Intake lobe smoothening. Hard to explain the details, but higher values - more smoothening and vice versa.
      "exhaust_sigma":1.0,              Exhaust lobe smoothening. Hard to explain the details, but higher values - more smoothening and vice versa.
      "intake_base_mult":1.5,           Intake lobe base radius multiplier, for visualization purposes only.
      "exhaust_base_mult":1.5,          Exhaust lobe base radius multiplier, for visualization purposes only.
      "intake_cos":false,               Setting it to "true" changes intake lobe generation from exponential to cosine, yielding different shape and behavior.
      "exhaust_cos":false,              Setting it to "true" changes exhaust lobe generation from exponential to cosine, yielding different shape and behavior.
      "equal_base_radius":true,         Setting it to "False" allows you to have different lobe base radius in the visualization. 
      "ramp_steepness":0,               Changes ramp steepness between 0.00 lift and "at_lift" value lift. 0 is disabled, 10 is maximum recommended. Can be used to decrease duration at 0.00 while keeping "at_lift" duration the same.
      "ramp_position": 0,               Changes the 0.00-"at_lift" ramp shape. 
      "lift_significant_fraction":100   Wish I could explain it to you, just take a look at what it does in the GUI.
      "roller_tappet_radius": 0.1       Roller tappet radius relative to cam base radius. Changes valve lift profile while keeping lobe the same. 
   }
}
*/


## Support

If any issues arise - contact Archangel Motors first. Please do not contact Oror regarding issues unless instructed to do so by Archangel Motors.
