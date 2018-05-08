# Reviewer 1
### Summary
Given the number of configuration options that many modern software systems offer, it is difficult to determine what configuration to use in a given environment. Prior work has identified the potential to use transfer learning to learn to predict how a software system will perform in a target environment, based on how it performs in a source environment.

This paper extends the prior work by proposing a method to identify "bellwether" environments: environments which can serve as source environments to train transfer learners for performance optimization of a given software system in a target environment. It then proposes BEETLE, a novel transfer learning algorithm which uses a bellwether source environment to train a transfer learner. The learners are claimed to find near-optimal configurations of a software system for the target environment.

The paper describes BEETLE and the results of evaluating it on five open-source software systems, comparing its performance to that of two state-of-the-art transfer learning systems. The results suggest that software system configurations identified for the target environment by the transfer learner are closer to optimal than those identified using two other state-of-the-art methods.

### Detailed evaluation

##### 1.1 First, what are the prerequisites for selecting a bellwether environment, and how does selection occur?

There are no prerequisites for selecting the bellwether environment. While conducting our experiments, we were not biased while choosing the environments instead, we used our domain knowledge to choose environments which models the real-world scenarios.


##### 1.2 Section 5 suggests that some number of previously optimized environments running optimal configurations of a software system, A, must exist, and that these are used to find a bellwether environment suitable for training a transfer learner that can find a near-optimal configuration of A for a new environment. How would I know if I had an adequate set of such environments, apart from checking to see if there is a bellwether environment which enables prediction of near-optimal configurations of the existing environments?

There are two basic scenarios where BEETLE can be used:
A user (in the real-world) maybe N different environments (workload + hardware + software version) to optimize and a limited budget (in terms of total number of measurements/execution). In such cases, BEETLE can be used to find a bellwether environment. Once the bellwether environment is identified, more samples is collected/measured to build a predictor which can be used to find the near-optimal configurations for the rest of the environments.
A user may have already collected/measures the samples from M different environments. If the user now need to find the near-optimal configuration for a new environment, instead of performing a cold start, the user can find BEETLE to find the bellwether to find the best configuration. Please note that the bellwether can change as measurements from new environments are added. In such case, BEETLE should be used in an active learning setting (discover, transfer and redo---as prescribed in [http://tiny.cc/fse2018_rebuttal_2]).

1.3 On this topic, I was particularly confused about how similar to a new target environment the existing source environments need to be. Can it actually be the case that a bellwether good enough to predict near-optimal configurations for one set of environments is also good enough to predict a near-optimal configuration for a really different environment? E.g., would I expect to be able to find a bellwether for A running on an edge device, like a NUC, with Windows, when all of my existing environments on which A is running are Cloud servers running various flavors of Linux? Can the BEETLE approach identify bellwethers which are good for transfer learning over particular clusters of environments with some unspecified degree of similarity, but which cannot produce a near-optimal configuration for a sufficiently different environment? This is more what I would expect to see. Did the authors consider this issue?

Yes, we have considered such a scenario. One of the objectives of BEETLE, is to rank the available sources in terms of similarity, which is measured in terms of how close is best performing configuration of the source environment is to the target environment. Hence, given two clusters of environments, the similar environments will be ranked higher than the not-so-similar environments.

1.4 Second, what specific information do I (as a user) need to have or be able to compute about software system operating in source environments, apart from values of the chosen performance measure (y, according to Section 4)? How is y computed? E.g., does the user have to supply a function to compute it (e.g., by running an established benchmark)?

All the user needs to know are the various configuration options that can be tweaked. A user should also be able to execute a particular benchmark (with a configuration selected by BEETLE) to measure the performance corresponding to that particular configuration


On this topic, there were several important values that had to be specified or known for the bellwether selection algorithm and BEETLE (Figures 2 and 3), but I was not clear about all of them. Figure 2 was useful, but it did not have enough information:

1.5 I understand sampling very well and I know that sampling is occurring on a source environment (source.sample(step_size) from Figure 2, line 6), I am not sure _what_ is being sampled in the source environment. Is the performance metric (y) being measured as the sample() action?
Something else?
Configurations and the corresponding performance measure.

1.6 Moreover, I was confused by the code in Figure 2 vs this statement from section 5.1: "The process starts by sampling a small subset of the source environments. The size of the subset is controlled by a predefined parameter step_size..." However, Figure 2 (lines 5-6) specifically show the for-loop looping over ALL of the sources that were passed in as a parameter, and calling sample(step_size) on _each one_.

Thank you for pointing this out. We meant “The process starts by sampling a small subset of the configuration from the source environments.”. Figure 2 is accurate.

    * I do not know how to define appropriate values for step_size, lives, budget, and thres, but these all affect the algorithms' performance.
In the paper, the values of the hyper-parameters were set by performing a Hyper-parameter sensitivity study (as discussed in Figure 11).

    *
Section 4 defines an "environment" as e = {h, w, v}. What is w, the workload, and how is it specified? (Based on the Dropbox URL provided in the paper for the more detailed version of Table 1, I'm guessing it is related to the inputs on which the performance was evaluated, but I would like the paper to explain this more clearly.)


As a note, I would have found it extremely valuable to have seen a simple, running example that showed, for some software system, running in a small number of environments, how the generic algorithm in Figure 2 would pick an appropriate bellwether for training a learner for a new environment. It should show a trace through the generic algorithm, supplying the details of parameter values and information that comes back from function calls (such as sample()) and what pieces of information are used at each step. I don't think it would have to take a lot of extra room. It would be fine as additional material reachable by a link in the paper. Having the complete code for bellwether selection and BEETLE might also have been valuable.

The source code for BEETLE can be found in http://tiny.cc/bw_source.

Third, what features does BEETLE use to train the transfer learner? Section 4 states that transfer learners use <c, y>. How is c represented, feature-wise?
A configuration ‘c’ is a vector of all the configuration options while y is the performance corresponding to that configuration.

Is it one composite feature, or is it broken down into individual configuration option values (or something else)?

As part of this question, I wasn't clear whether or how environmental variables are represented or accounted for. The only representation of "environment" provided was e = {h, w, v}. However, many environmental characteristics affect performance, just like software configuration parameters do. For example, if my software system is Apache Lucene, I must ensure that an appropriate JVM is installed in an environment before I can run Lucene in that environment. The JVM is not a configuration option on Lucene; it's part of the "environment" that Lucene expects. Selection of one version of a JRE vs. another can have profound performance impact. But this environmental variable isn't obviously represented in the definition of environment ({h, w, v}); hence, it looks to me as though I should treat two environments as being identical if they have the same hardware,
workload, and software version, but not the same JVM version, or database version, etc. This seems odd to me. I may be misunderstanding this definition. I would really appreciate some insight into how environmental "factors" are represented and used, if at all--and if not, why?

We wanted to use the same terminology of the ASE paper[http://tiny.cc/fse2018_rebuttal_3]. In the Storm experiment, we use different JVM versions (since different versions of Storm uses JVM). We can include other factors to describe the environment, but we only use workloads, hardware, and software versions to conduct of our experiments---something which the developers care about. However, a user can use any external factor/s as environment.

Other issues, including pointers to the typos, grammatical issues, etc., are included below.


Detailed Comments
[TBD]

----------- Strengths and weaknesses -----------
+ An interesting paper on an important topic
+ The problem and solution are both well motivated
+ The solution approach seems promising and potentially valuable for a pervasive problem
+ The evaluation seemed appropriate and answered relevant questions

- The paper is more difficult to understand than necessary, due to many typos, grammatical issues, organizational and presentation issues, and sloppiness
- No replication package was provided, and only a subset of the code to identify bellwethers and train/test them was provided
- Not fully clear prerequisites for successful use
- Unclear what features are being used to train the predictive models; hence, it is difficult to evaluate the results based on the 5 test subject systems

----------- Questions to the authors -----------
TBD

----------------------- REVIEW 2 ---------------------
PAPER: 178
TITLE: Transfer Learning with Bellwethers to find Good Configurations
AUTHORS: Vivek Nair, Rahul Krishna, Tim Menzies and Pooyan Jamshidi

Overall score: -1 (weak reject)

----------- Summary -----------
The paper extends the work of Sigemund et al. and Jamshidi [13,14, 19-21,37,41, 49]. The idea is generally to have a sampling based approach to identify “and transfer” performance characteristics of product lines. The solution assumes that there are so-called bellwether environments that help with transfer learning.

----------- Detailed evaluation -----------
The presented work is sound and novel with respect to applying bellwethers to transfer learning. In this regard, the paper provides an incremental improvement to the research on performance prediction of software product lines with multiple configurations and a huge configuration space.

The motivation of the paper is nice.

I have some problems with understanding the problem formalization in section 3. Specifically what is the software version $v \in V $ is.
We can improve the problem formalization in the next version of the paper. Version in our context means the state of the code base at a certain point in time. For example, the latest version of Storm is 1.2.1.
Furthermore, the section on “Configurations” is very confusing, here I would start with the Cartesian product $C$ and then describe the options.

For performance the authors and all predecessors assume [13,14, 19-21,37,41, 49] that performance can be measured directly with a median value as a measure for a central tendency and confidence intervals or similar spread measures. However, in practice, performance is analysed via performance curves and one has to identify which part of the import domain is triggering which parts of the performance curve.

The papers is very limited takes an environment, the hardware and the software version whatever it is into account. Here I would have directly two question for the rebuttal.

Q1: How would the approach be able to integrate performance curves for practical usage of the bellwether transfer learning?
We do acknowledge that prior work used MMRE and reported the median and variance across multiple runs. However, we realized that in the real-world scenario, MMRE does not provide the user any information. So rather than using MMRE, we are using a new performance measure called NAR, which is the normalized distance from the optimal configuration (lower the better). This is party inspired by FSE’17 paper [http://tiny.cc/fse18_rebuttal_1]. A low (median) NAR value represent that the selected source can find the configuration which is closer to part of the curve closer to optimal configuration. As the value grows larger, it moves away from the optimal configuration.



Q2: How does the overly simplified environment characterisation affect the validity of the results?
Maximize difference;
Previously used; and
Costs time/money.

Yes I am aware the original papers by Sigemund and Jamshidi suffer from the same limitations. However since this is an advanced research paper and not just an early ideas paper, a deeper discussion and maybe method would help.

In table 1 the numbers for environments are extremely small. I assume that the low number of hardware options is fine since there will be a direct relation to the performance and thus this will be perfect for transfer learning. However, specifically SAT solver inputs for instance can be very diverse leading to massive performance differences. However guessing from section 7.1 I could not determine what specifically the environment is for each system.

Specifically for SAT solver we used different SAT problems of varying complexity because we acknowledge the

One of the authors performed comprehensive studies with different complexities.


Q3:Could you explain what the environments in the experimental setup in table 1 and how you have selected them.

@Rahul.

After reading the experimental setup and table 1 I think I got an idea what you mean by software version. This could have been made more clear earlier in the paper

Based on the Benchmark provided the results show the Bellwether exist, that you can discover them with limited sampling, that the Beetle approach with Bellwethers outperforms traditional non-transfer learning and state of the art transfer learning approaches.

The discussion provides insights in tuning of hyper parameters, the ineffectiveness of bellwethers in certain circumstances as well as their applicability to other domains. From a research perspective this is enough.

However one big question remains:
Q4 What is the effect of beetle on the day to day business of a software engineer that aims to optimize the performance of its configurable system.

There are two basic scenarios where BEETLE can be used:
A user (in the real-world) maybe N different environments (workload + hardware + software version) to optimize and a limited budget (in terms of total number of measurements/execution). In such cases, BEETLE can be used to find a bellwether environment. Once the bellwether environment is identified, more samples is collected/measured to build a predictor which can be used to find the near-optimal configurations for the rest of the environments.
A user may have already collected/measures the samples from M different environments. If the user now need to find the near-optimal configuration for a new environment, instead of performing a cold start, the user can find BEETLE to find the bellwether to find the best configuration. Please note that the bellwether can change as measurements from new environments are added. In such case, BEETLE should be used in an active learning setting (discover, transfer and redo---as prescribed in [http://tiny.cc/fse2018_rebuttal_2]).

Furthermore:  
Q5 What effect would beetle have on dynamic product lines?
@Pooyan
We have recently tried transfer learning for configuration tunning

----------- Strengths and weaknesses -----------
+ the paper improves the current state of the art
+ the paper has a simple yet fascinating fascinating idea and I am glad that the result provide evidence the this idea works
-       The practical impact of the research can be described better
-       The problem formalization can be improved and adapted to a real software engineering context

----------- Questions to the authors -----------
Q1-Q5 are provided in the detailed evaluation.

----------------------- REVIEW 3 ---------------------
PAPER: 178
TITLE: Transfer Learning with Bellwethers to find Good Configurations
AUTHORS: Vivek Nair, Rahul Krishna, Tim Menzies and Pooyan Jamshidi

Overall score: 1 (weak accept)

----------- Summary -----------
The authors evaluate whether Bellwether environments exist that can more accurately predict (using transfer learning) the optimal configuration of a system in a different environment. An empirical study on 5 open source projects and a variety of hardware, workloads and project versions shows how only 10% of performance measurements are required to find good Bellwether environments that, together with transfer learning, are able to perform as well as 2 state-of-the-art transfer learning approaches.

----------- Detailed evaluation -----------
I appreciate the topic of this paper, since it indicates a timely problem gathering substantial interest from industry. The authors also performed a large empirical study, as witnessed by the cost figures plotted in Fig. 1.

The paper's research questions might need reformulation. In particular, the first one ("RQ1: Does there exist a Bellwether Environment?") almost trivially could be answered as "yes", since one could not expect all environments to exhibit identical performance, there will always be some environment that is able to make better predictions (although, by chance, the findings do show such an example, see below). Hence, I would rephrase this RQ in terms of "How good do Bellwether environment predictions perform?").

I did not fully agree with the decision to include workload and software version in the definition of "environment", since traditionally the environment of an application does not include the application itself nor the usage scenarios of end users on the application. Different workloads exercise different features of the application, while the underlying run-time environment of the system would still be the same.

Overall, the paper does a good job explaining the approach, however there are a number of decisions that are not well discussed. If I understood correctly how BEETLE works, the search for Bellwether environments mostly is random, hoping to find such environments as soon as possible using random sampling.
3.1 Did the authors consider a more focused search metaheuristic?
No, we have not consider any other search heuristic, but it can be in practical setting, an user can use any sampling method of choice.

3.2 Why is the paper limited to boolean options only, and how does BEETLE generalize to non-boolean options?
Yes, BEETLE can generalize to non-boolean options.

3.3 Why is MMRE not applicable in this paper?
Our objective is to find the near optimal configuration and hence is an optimization problem (which does not require an accurate model [http://tiny.cc/fse18_rebuttal_1]). To correctly evaluate the closeness of the predicted optimal to the actual optimal, we use NAR---which is the normalized absolute residual of the performance scores.

(cf. "While this typical in performanceactualprediction, our objective is tofind the near-optimal configurations or performance optimization.For this, measures similar to MMRE is not applicable").

I also did not understand the scaling factor and kernel function used for Jamshidi et al.'s approach.

3.4 Storm are ranked equally, i.e., no environment performs better than another one to build prediction models
In this case, all the environments are bellwethers based on the median NAR values. But, we do agree  that this might be caused due to few environments.

It seems like all environments for Storm are ranked equally, i.e., no environment performs better than another one to build prediction models. In that case, there is *no* Bellwether environment, if we use the definition used by the paper. Or is this just a case of bad luck, where the environments used in the study are too limited and similar? The conclusion "There exist environments in each subject system,which act as bellwether environment " should be revised based on this.

While I like the idea to repeat runs 30 times to reduce noise in the performance numbers, I got confused in the discussion of the Scott-Knott test, which is said to rank treatments from best to worst. Are these the 30 runs that are being ranked, or are runs of different environments compared to each other based on each environment's Scott-Knott results? Maybe I misunderstood what the 30 repeated runs are used for?

3.5 Does the median NAR in the figures refer to the median across 30 runs?
Yes.

I liked the discussion in Fig. 11 on how the available budget and number of lives impact the performance of the results. It would be interesting to see similar discussions for other parameters, as hinted at in the threats to validity.

3.6 For the analysis in Fig. 8, I was wondering if for BEETLE different percentages of data are being used, or that still only 10% is used (which was the optimal percentage)?
Only 10% is used.

The paper concludes that in 4 out of 5 systems BEETLE performs at least as good as the state-of-the-art transfer learner approaches. I was wondering whether the improvements for Spear, Sqlite and Storm are in fact significant, since one could also interpret the results as Jamshidi et al.'s approach performing at least as good as BEETLE in 4 out of 5 of the systems.

3.7 Why do both approaches perform similarly well (at least from the accuracy point of view)?
The method proposed by Jamshidi et al. is effective as well as complicated. Our objective is to show that a very simple method like BEETLE can be very effective. Also note that, from accuracy/effectiveness point of view BEETLE is similar to the method proposed by Jamshidi et al, which use fewer number of measurements as well as requires less training time (since our method does not use Gaussian Processes, which does not scale for higher dimensions).


Page 1: " evaluate our approach with 61 scenarios based on 5 software systems and demonstrate that BEETLE is beneficial in all cases.":
Where does the number 61 come from?
Sum of column, |E|, of Table 1. This sums up to 57, but we also measure latency of Storm for 4 different workloads and hence 61.

Page 1: " source agnostic complex transfer learners.":
What about simple, source-dependent learners, or are those not possible?
Most transfer learners proposed does not consider choosing the appropriate source as a transfer learning strategy. The method proposed by Valov et al. might be considered as a simple, source-dependent learner.

Page 4: "In oursetting, we use the number of measurements as a proxy ":
I.e., the number of execution runs, or the actual time measurements?
Number of execution runs.

Table 1: It seems like |E| does not always equal |H| x |W| x |V|, why is this?
It is a typo, which we will fix it in the next version of the paper.

Page 6: "e difference between theactual and predicted optimal configuration is normalized to thedifference between the actual best and worst configurations":
This means that the measure can actually go outside the [0,1] interval if f(c*) goes outside range?
The measure ranges between [0, 100] and f(c*) will not go out of range.

Fig. 9: Does "Baseline" for SAC refer to "BEETLE"?
Yes.

----------- Strengths and weaknesses -----------
In favour:
 * timely research topic
 * large empirical study

Against:
 * how to interpret findings compared to Jamshidi et al.?

----------- Questions to the authors -----------
* Can the authors clarify the use of Scott-Knott tests?
  * How should one interpret the accuracy results of BEETLE compared to Jamshidi et al.?

------------------------------------------------------
