# ir
4. EMPIRICAL STUDY
Objective: study the proposed RTB feedback control mechanism
	1. Evaulation set up
          - dataset: iPinYou DSP(
             ad log data from 9 campaigns during 10 days in 2013, which consists of 64.75M bid records, 19.50M impressions, 14.79K clicks and 16K Chinese Yuan (CNY) expense)
             each row consists of three parts:  : (i) The features for this auction, e.g., the time, location
							(ii) The auction winning price
							(iii) The user feedback on the ad impression, i.e., click or not
                     
      -  Evaluation Protocol
         pass the feature information to our bidding agent -> new bid based on the CTR prediction and other parameters in Eq   -> compare the generated bid B1' with the logged actual auction winning price  B1:
     IF  B1' > B1,   bidding agent has won this auction: click(positive outcome) ;  no click(negative outcome  ) 
                                                                                              the control parameters are updated every 2 hours (as one round)
   users' contexts should not be change:   under the same context if the advertiser were given a different or better bidding strategy or employed a feedback loop, whether they would be able to get more clicks with the budget limitation.
   
   - Evaluation Measures
     error band: the ±10% interval around the reference value
    rise time: how fast the controlled variable will get into the error band
    settling time: e how fast the controlled variable will be successfully restricted into the error band
     two control accuracy measures:
     overshoot:e the percentage of value that the controlled variable passes over the reference value
    RMSE-SS: the root mean square error between the controlled variable value and the reference value
    stability(SD-SS):  calculating the standard deviation of the variable value after the settling

 - Empirical Study Organisation
   focused on 2 KPIs: eCPC   AWR
   -  2 whether the proposed feedback control systems are practically capable of controlling the KPIs
  -  3 control difficulty with different reference value settings
  -  4 PID controller and investigate its attributes on settling the target variable  -  5 leverage the PID controllers as a bid optimisation tool and study their performance on optimising the campaign’s clicks and eCPC across multiple ad exchanges
  - 6 tuning and online update

	2.  Control Capability

  -   tune the control parameters on the training data to minimise the settling time
  -   work on test data and then observe the performance
   
   (i) all the PID controllers can settle both KPIs within the error band
   (ii) The WL controller on eCPC does not work that well on test data, even though we could find good parameters on training data  (WL controller tries to affect the average system behaviour through transient performance feedbacks while facing the huge dynamics of RTB)
    (iii) For WL on AWR, most campaigns are controllable while there are still two campaigns that fail to settle at the reference value
  (iv) Compared to PID on AWR, WL always results in higher RMSE-SS and SD-SS values but lower overshoot percentage
  (v) the campaigns with higher CTR estimator AUC performance normally get shorter settling time.
 summary: PID controller outperforms the WL controller in the tested RTB cases
 (reason: the integral factor in PID controller helps reduce the accumulative error (i.e., RMSE-SS) and the derivative factor helps reduce the variable fluctuation (i.e., SD-SS). And it is easier to settle the AWR than the eCPC. This is mainly because AWR only depends on the market price distribution while eCPC additionally involves the user feedback, i.e., CTR, where the prediction is associated with significant uncertainty.)


 3 Control Difficulty
  compared to 2, adding higher and lower reference values in comparison
  investigate the impact of different levels of reference values on control difficulty
 Result: 
 -  the average settling time are reduced as the reference values get higher
Reason: generally the control tasks with higher reference eCPC and AWR are easier to achieve because one can simply bid higher to win more and spend more.
 - For higher reference: the control signal does not bring serious bias or volatility, which leads to the lower RMSE-SS and SD-SS. 
Reason: the higher reference is closer to the initial performance value
 - the reference value which is farthest away from the initial value of the controlled variable brings the largest difficulty for settling, both on eCPC and AWR
Reason: advertisers setting an ambitious control target will introduce the risk of unsettling or large volatility

4  PID Settling: Static vs. Dynamic References
One may empirically adjust the reference value in order to achieve the desired reference value
1 Dynamic Reference Adjustment Model
simulate the advertisers’ strategies to adaptively change the reference value of eCPC and AWR under the budget constraint: new reference xr(tk+1) after tk:
Results:
For both eCPC and AWR control, the dynamic reference takes an aggressive approach and pushes the eCPC or AWR across the original reference value
Reason: when the performance is lower than the reference, then higher the dynamic reference to push the total performance to the initial reference more quickly
the dynamic reference fluctuates seriously when the budget is to be exhausted soon
when there is insufficient budget left, the reference value will be set much high or low by Eq. (20) in order to push the performance back to the initial target.
compare the settling cost, which is the spent budget before settling.

  (i) for eCPC control, the dynamic-reference controllers do not perform better than the static-reference ones; 
  (ii) for AWR control, the dynamic-reference controllers could reduce the settling time and cost, but the accuracy (RMSE-SS) and stability (SD-SS) is much worse than the static-reference controllers.
This is because the dynamic reference itself brings volatility


5 Reference Setting for Click Maximisation
bid requests usually come from different ad exchanges where the market power and thus the CPM prices are disparate.
have shown that given a budget constraint, the number of clicks is maximised if one can control the eCPC in each ad exchange by settling it at an optimal eCPC reference for each of them, respectively
build a PID feedback controller for each of its integrated ad exchanges
Multiple: multi-exchange eCPC feedback control
Uniform: test a baseline method which assigns a single optimal uniform eCPC reference across all the ad exchanges

Results:
(i) the feedbackcontrol-enabled bidding strategies uniform and multiple significantly outperform the non-controlled bidding strategy none in terms of the number of achieved clicks and eCPC.
(ii) By reallocating the budget via setting different reference eCPCs on different ad exchanges, multiple further outperforms uniform on 7 out of 8 tested campaigns.
(iii) On the impression related measures, the feedback-control-enabled bidding strategies earn more impressions than the non-controlled bidding strategy by actively lowering their bids (CPM) and thus AWR, but achieving more bid volumes.
( by allocating more budget to the lower valued impressions, one could potentially generate more clicks)

6 PID Parameter Tuning
Parameter Search:  For every update, we fix one parameter and shoot another one to seek for the optimal value leading shortest settling time, and the line searching step length shrinks exponentially for each shooting.
Setting φ(t) Bounds:  setting up upper/lower bounds of control signal φ(t) is important to make KPIs controllable
Online Parameter Updating:   after initialising the PID parameters using training data, we re-train the controller for every 10 rounds (i.e., before round 10, 20 and 30) in the test stage using all previous data with the same parameter searching method as in the training stage.
after the 10th round (i.e., the first online tuning point), the online-tuned PIDs manage to control the eCPC around the reference value more effectively than the offline-tuned one, resulting shorter settling time and lower overshoot



5. ONLINE DEPLOYMENT AND TEST
Conducted at BigTree DSP
last 6-week bidding log data in 2014 as the training data for tuning PID parameters. A three-fold validation process is performed to evaluate the generalisation of the PID control performance, where the previous week data is used as the training data while the later week data is used for validation.

Compared with the offline empirical study, the online running is more challenging: 
(i) all pipeline steps including the update of the CTR estimator, the KPI monitor linked to the database and the PID controller should operate smoothly against the market turbulence;
(ii) the real market competition is highly dynamic during the new year period when we launched our test; 
(iii) other competitors might tune their bidding strategies independently or according to any changes of their performance after we employed the controlled bidding strategy


7. CONCLUSION
 proposed a feedback control mechanism for RTB display advertising, with the aim of improving its robustness of achieving the advertiser’s KPI goal:
mainly studied PID and WL controllers for controlling the eCPC and AWR KPIs
(i) Despite of the high dynamics in RTB, the KPI variables are controllable using our feedback control mechanism
(ii) Different reference values bring different control difficulties, which are reflected in the control speed, accuracy and stability
(iii) PID controller naturally finds its best way to settle the variable, and there is no necessity to adjust the reference value for accelerating the PID settling. 
(iv) By settling the eCPCs to the optimised reference values, the feedback controller is capable of making bid optimisation.
