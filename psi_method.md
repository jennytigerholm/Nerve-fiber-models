# Threshold Estimation

## Psi method 
The threshold estimation tests makes it possible to estimate psychometric functions with adaptive methods. 
The purpose and motivation for adaptive methods is to increase the effectiveness by which pasychometric 
functions are estimated, so this process becomes faster and ultimately results in better estimates as 
the effort required by the subjects becomes less. They achieve that by attempting to maximize the 
information gained from each presentation of a stimulus to the subject.

[Back to index](index.html)

## Description 

The psi method defines a space of possible psychmetric function defined by the the four papmeters: alpha (threshold), besta(slope rate), gamma (guessing rate), lambda (lapsing rate). The accuracy of the estimated threshold will depend of the size of vectors alpha and beta therefore choose these valuble with care. 
The alpha vector is scaled with the interger paramter "Imax", wich difines the maximum allowed current intestity for that threshold estimation.
Also the Imax value need to be defined with care since electrical  stimulation tresholds may differs significantly by different electrical stimulation shapes, 
subjects and placement of the electrode on the body. A god stragegy is to prior to the Psi method roughly estimate the threshold by the UP/DOWN method (see thew example below). 
The Psi algorithm uses an look up table for the probablitiy of preciving a specific stimulation for all possible phucametric function defines by 
the four phycometric function.  

The first stimulation is choosen as the median value of the alpha vector then following stimulation intensity id choosen to maixize the expected 
informatio gained disregardng if the subject precives the stimulation or not. The algoritm will continoue untill the the number of maximum trials has 
been preformed, which is predefined in the protocol file.

[Back to index](index.html)

## Required instruments

The Thrshold Estimation test needs the following instruments to be defined in the expriment definition file (*.expx) for the experiment:

| Name      |  Interface        | Usage |
|-----------|-------------------|-------|
|Button     | IButton           |Is used for the subject to indicate whether he or she can feel the stimulation|
|Stimulator | IAnalogStimulator |Is used for delivering the stimuli to subject|

In the xml snippet below it is illustrated how these instruments can be defined in experiment definition file (*.expx) for all Threshold Estimation tests in a protocol:

[Back to index](index.html)

## Estimation Algorithms

### Psi Estimation Algortihm

```xml

<?xml version="1.0" encoding="UTF-8"?>
<protocol xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:xsd="http://www.w3.org/2001/XMLSchema"
          xsi:schemaLocation="http://labbench.io http://labbench.io/schema/current/protocol.xsd"
          name="Example_prot"
          version="1.0.0">
          <description>
            This is a Example protocol.
                    Two elctrical test stimulation threshold will be evaluated. 
                    The stimulation is preformed with random inersimulus interavls beween 2000ms to 3000ms
                    First a rough estimation of the treshold is preforemed with the UP/DOWN method in order to estimat the Imax value.
                    Secondly the Psi method is estimation the threshold by 30 trials.  
          </description>

  <defines>
<!-- Stimuli -->

      <define name="T1" value="0.5"/
      <define name="T2" value="1"/>


    <!-- Method of Limits. For more information regaring these paramter see the section in the manual for the Up-and-down method -->
    <define name="Istart" value="0.025"/>
    <define name="Naverage" value="1"/>
    <define name="Ndiscard" value="1"/>
    <define name="Ntest" value="2"/>
    <define name="Pdecrease" value="0.2"/>
    <define name="Pmin" value="0.05"/>
    <define name="Pstep" value="0.2"/>
    <define name="ReversalRule" value="3" />
    <define name="StartIntensity" value="0.025" />
    <define name="StepSize" value="0.25" />
    <define name="StepSizeReduction" value="0.5" />
    <define name="MaxStepSizeReduction" value="0.25"/>
    <define name="SkipRule" value="0"/>
    <define name="StopRule" value="2"/>
    
    <!-- Psi Method -->
    <define name="Trials" value="30"/>
    <define name="Imult" value="4"/>
    <define name="alphaX0" value="0.05"/>
    <define name="alphaN" value="200"/>
    <define name="intensityX0" value="0.05"/>
    <define name="intensityN" value="100"/>


  </defines>
<tests>

<!-- Starting values. This section generates the starting values for defining the maximum intensity-->
<threshold-estimation ID="PsiS" name="generating start values">
  <update-rate-random min="2000" max="3000" /> <!-- interval in ms between the stimuli-->
  <dependencies>
    <dependency ID="TEST"/>
  </dependencies>
  <yes-no-task />
  <channels>
    <channel ID="C01"
             channel-type="single-sample"
             trigger="1"
             channel="0"
             name="Test stimulus 1"
             Imax="Stimulator.Imax">
      <up-down-method down-rule="1"
                      up-rule="3"
                      max-intensity="1"
                      min-intensity="0"
                      skip-rule="0"
                      start-intensity="0.005/Stimulator.Imax"
                      step-size-up="0.25"
                      step-size-down="0.25"
                      step-size-type="relative"
                      terminate-on-limit-reached="true"
                      max-step-size-reduction="0.1"
                      step-size-reduction="StepSizeReduction"
                      stop-criterion="reversals"
                      stop-rule="2">
        <quick alpha="0.5" beta="1" lambda="0.02" gamma="0.0" />
      </up-down-method>
      <combined>
            <!-- test pulse -->
           <pulse Is="Is" Ts="T1" Tdelay="0"/>
           
           <!-- charge balancing-->
           <pulse Is="-Is*0.2" Ts="T1*5" Tdelay="T1"/> 

       </combined>
    </channel>

    <channel ID="C02"
             channel-type="single-sample"
             trigger="1"
             channel="0"
             name="Test stimulus 1"
             Imax="Stimulator.Imax">
      <up-down-method down-rule="1"
                      up-rule="3"
                      max-intensity="1"
                      min-intensity="0"
                      skip-rule="SkipRule"
                      start-intensity="0.005/Stimulator.Imax"
                      step-size-up="StepSize"
                      step-size-down="StepSize"
                      step-size-type="relative"
                      terminate-on-limit-reached="true"
                      max-step-size-reduction="MaxStepSizeReduction"
                      step-size-reduction="StepSizeReduction"
                      stop-criterion="reversals"
                      stop-rule="StopRule">
        <quick alpha="0.5" beta="1" lambda="0.02" gamma="0.0" />
      </up-down-method>
      <combined>
          <!-- test pulse -->
           <pulse Is="Is" Ts="T2" Tdelay="0"/>
           
           <!-- charge balancing-->
           <pulse Is="-Is*0.2" Ts="T2*5" Tdelay="T2"/> 
       </combined>
    </channel>
  </channels>
</threshold-estimation>

<!-- Psi method  1 -->
    <threshold-estimation ID="PTT1" name="PTT five pulse shapes (Psi Estimation) 1">
      <update-rate-random min="2000" max="3000" /> <!-- interval in ms between the stimuli-->
      <dependencies>
        <dependency ID="PsiS"/>
      </dependencies>
      <yes-no-task />

      <channels>
        <!-- input 1-->
        <channel ID="C01"
                 channel-type="single-sample"
                 trigger="1"
                 channel="0"
                 name="Test stimulus 1"
                 Imax="Imult * PsiS['C01'] if Imult * PsiS['C01'] &lt; Stimulator.Imax else Stimulator.Imax">
          <channel-dependencies>
          </channel-dependencies>
          <psi-method number-of-trials="Trials">
            <quick alpha="0.5" beta="1" lambda="0.02" gamma="0.0" />
            <beta type="linspace" base="10" x0="-1.2041" x1="1.2041" n="20"/>
            <alpha type="linspace" x0="alphaX0" x1="1" n="alphaN" />
            <intensity type="linspace" x0="intensityX0" x1="1" n="intensityN" />
          </psi-method>
´             <combined>
            <!-- test pulse -->
           <pulse Is="Is" Ts="T1" Tdelay="0"/>
           
           <!-- charge balancing-->
           <pulse Is="-Is*0.2" Ts="T1*5" Tdelay="T1"/> 
       </combined>
        </channel>

        <!-- input 2-->
        <channel ID="C02"
                 channel-type="single-sample"
                 trigger="1"
                 channel="0"
                 name="Test stimulus 2"
                 Imax="Imult * PsiS['C02'] if Imult * PsiS['C02'] &lt; Stimulator.Imax else Stimulator.Imax">
          <channel-dependencies>
          </channel-dependencies>
          <psi-method number-of-trials="Trials">
            <quick alpha="0.5" beta="1" lambda="0.02" gamma="0.0" />
            <beta type="linspace" base="10" x0="-1.2041" x1="1.2041" n="20"/>
            <alpha type="linspace" x0="alphaX0" x1="1" n="alphaN" />
            <intensity type="linspace" x0="intensityX0" x1="1" n="intensityN" />
          </psi-method>
            <combined>
          <!-- test pulse -->
           <pulse Is="Is" Ts="T2" Tdelay="0"/>
           
           <!-- charge balancing-->
           <pulse Is="-Is*0.2" Ts="T2*5" Tdelay="T2"/> 
       </combined>
      </channel>
  </threshold-estimation>

  </tests>
</protocol>

     
```

elow is a description of each individual parameter in the Psi algorithm method in the test setup.

|Parameter|Type|Description|
|----------------|----|-----------|
|alpha      |vector   | possible threshold for the phycometric function space  |       
|geta       |vector   | possible slope rate for the phycometric function space |
|gamma      |integer  | guessing rate for the phycometric function space   |
|lambda     |integer  | lapsing rate for the phycometric function space   |
|intensity  | vector  | Normalized possible stimulations intensities. Note that this vector is scaled with the Imult to generate the possible stimulations intensity|
|Trials     | integer | Number stimulation before the algorithm stops|
|Imax       | integer | The maximum intensity|

Below is a description of each individual parameter in the UP/DOWN algrorithm in the test setup.

|Parameter|Type|Description|
|----------------|----|-----------|
| down-rule      | integer | How many stimulations that fallen below the threshold before the intensity will  increasing |
| up-rule        | integer | How many stimulations that has to be above the threshold before the intensity will be decreasing |
| max-intensity  | double   | Normalized maximal stimulation intensity |
| min-intensity  | double | Normalized minimal stimulation intensity |
| skip-rule      | integer | Number of initial reversals that are skipped and not included in the calculation of the threshold |
| start-intensity | double | Normalized starting intensity |
| step-size-up    | double | Initial upwards step size, without factoring in step size reduction |
| step-size-down  | double | Initial downwards step size, without factoring in step size reduction |
| step-size-type | Enum (absolute, relative) | If absolute the step-size-up/step-size-down is in amperes, otherwise it is relative to current stimulation intensity |
| terminate-on-limit-reached | true/false | Will the test terminate if the max-intensity is reached |
| max-step-size-reduction | double | maximal step size reduction, before the reduction in step size saturates and will not be come any smaller (i.e. if set to 0.2 the step size will maximally be reduced to 20% of its initial value) |
| stop-criterion | Enum (trials, reversals) | If set to trials the test will stop after a set number of stimulations, regardless of whether a threshold estimate has been achieved or not. If set to reversals it will terminate after a set number of reversals that the threshold has been reached and not reached. |
| stop-rule | integer | Number of stimulations (stop-criterion = trials) or reversals (stop-criterion = reversals) when the test will terminate |

[Back to index](index.html)

### Psi Estimation Algorithm

beta type="linspace" base="10" x0="-1.2041" x1="1.2041" n="20"/>
            <alpha

[Back to index](index.html)
