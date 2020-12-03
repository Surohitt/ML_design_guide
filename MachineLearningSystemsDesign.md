## Contents:
1. [Introduction](#introduction)
2. Complex Models Erode Boundaries
    1. Entanglement
    2. Correction Cascades 
    3. Undeclared Consumers
3. Data Dependencies Cost More than Code Dependencies
    3.1 Unstable Data Dependencies 
    3.2 Underutilized Data Dependencies 
    3.3 Static Analysis of Data Dependencies 
4. Feedback Loops 
    4.1 Direct Feedback Loops
    4.2 Hidden Feedback Loops
5. ML-Systems Anti-Patterns
    5.1 Glue Code 
    5.2 Pipeline Jungles 
    5.3 Dead Experiment Code Paths
    5.4 Abstraction Debt
    5.5 Common Smells
6. Configuration Debt
7. Dealing with Changes in the External World
    7.1 Fixed Thresholds in Dynamic Systems 
    7.2 Monitoring and Testing
8. Other Areas of ML-related Debt
    8.1 Data Testing Debt
    8.2 Reproducibility Debt
    8.3 Process Management Debt 
    8.4 Cultural Debt
9. Conclusions: Measuring Debt and Paying it Off    

## 1. Introduction

This document is to serve as a general guide for when developing and Machine Learning system/tool. The practices here are in accordance with the paper titled *Hidden Technical Debt in Machine Learning Systems* published by google in 2015. This paper has served as a source of truth for many organizations looking to scale their Machine Learning products and systems and can offer resolutions when there is ambiguity in specifications. 

**Why is this important?** Great question. In typical software development, there are a lot of references to *technical debt*. This relates the pace of development to the potential long term costs of accumulated and calculated oversights. Given that Keela operates under an agile environment, this is unavoidable. While technical debt isn't necessarily a bad thing (as with most forms of debt), *Hidden Technical Debt* is.

## 2. Complex Models Erode Boundaries

Since ML is typically required to solve tasks that are inherently difficult to express in traditional logic, it's difficult to enforce strict abstraction boundaries for ML systems by prescribing specifically intended behaviour.

### 2.1 Entanglement

When inputting a series of features into an ML algorithm, we begin to introduce a degree of entanglement. Consider the feature set $x_1,...x_n$. If we were to change the input distribution of $x_1$ we change the way the weights of our system or $n-1$ features are used. This holds true even if we add a new feature or even remove a feature. Even after re-training and allowing the model to adapt. This is often referred to as the *CACE* principle - *Change Anything Change Everything*.

**Solutions**:

1. **Isolate models and serve ensembles**. This works when we can decompose a problem into sub-problems and serve models as solutions as found in disjoint multi-class settings. This works well as the errors in component models are uncorrelated with each other. However, this results in strong entanglement where improving individual components may degrade the system overall.
2. **Detect changes in prediction behaviour as they occur**. Using a high-dimensional visualization tool allows engineers to quickly see the effects across many dimensions and slicings. Equally, monitoring metrics on a slice by slice basis may also be extremely useful $(\sigma^{2}, \mu, mse, accuracy)$.

### 2.2 Correction Cascades

Often, engineers will have a model $M_{a}$ that solves problem $A$. We then notice a new problem $A^{'}$  and logically conclude that a variation of $M_{a}$, say $M_{a'}$ would be a fast and easy solution. However, the corrected model $M_{a'}$ now has a dependency on the original model $M_{a}$.

Thus, any improvements to either model $M_{a}$  or $M_{a'}$  will have added future costs. Also, as we iterate and develop, we may also decide to add new models and dependencies i.e. $M_{a''}$. We now introduce compounding and cascading costs if we were to improve any of the original models. We call this problem an *improvement deadlock*, where any improvement leads to system-level detriments.

**Solutions**

1. Augment $M_{a}$ to learn corrections directly by adding features, thus being able to solve $A'$
2. Accept costs of creating a new and separate model, $M_{b}$ for problem $A'$.

### 2.3 Undeclared Consumers

This hidden technical debt is often referred to as *visibility debt* in classical software engineering. Undeclared consumers can be exceedingly dangerous. Users may take outputs of given models and use them as inputs into other systems. These users create tight couplings between systems and create hidden feedback loops. So any change to a model may affect the system as a whole and make improvements very difficult.

These users are difficult to detect unless the system is specifically designed to safe guard against this. 

## 3. Data Dependencies Cost More than Code Dependencies

 In traditional software engineering, *dependency debt* is where a core contributor to code complexity and any technical debt gained. Linkers and compilers typically identify where the dependencies are. However, no similar tools exist for data dependencies. This can lead to large dependency chains that are difficult to untangle.

### 3.1 Unstable Data Dependencies

Often, we have our models consume convenient data signals as input features. However, some of these signals may be unstable. E.g. They are likely to quantitatively or qualitatively change over time. This can happen in models or even data dependant lookup tables TF/IDF (term frequency in language modelling) scores. 

**Solutions**

- Versioned Copies. It's reasonable to create versioned copies of such data dependencies when considering stability of the source. Using the frozen version until new versions are vetted does introduce potential staleness and cost in terms of maintenance.

### 3.2 Underutilized Data Dependencies

Underutilized data dependencies are simply input signals that provide incremental modelling benefit. These dependencies only make our models more vulnerable to change. These dependencies can creep in several ways:

1. **Legacy Features**: A feature in our model is made redundant over time by other features but goes undetected. 
2. **Bundled Features**: Group of features are evaluated together and added to a models input due to time constraints. 
3. $**\epsilon$ -Features**: Incremental model improvement at the cost of complexity overhead
4. **Correlated Features**: Often, features can be correlated. Many Algorithms result in brittleness if world behaviour later changes the correlations. 

**Solutions**

- Evaluate feature dependencies using *leave-one-out* evaluations. These should run regularly to identify and remove unnecessary features.

### 3.3 Static Analysis of Data Dependencies

 Traditionally compilers and build systems perform static analysis of dependency graphs. Such tools for data dependencies are far less common but still remain essential for error checking, tracking down consumers and enforcing migration and updates. Such tools like automated feature management systems, which enables data sources and features to be annotated allow us to run automated checks to ensure all dependencies have the appropriate annotations and dependency tress can be fully resolved. This kind of tooling can make migration and deletion much safer in practice. 

## 4. Feedback Loops

As previously mentioned, feedback loops are dangerous and often lead to *analysis* of debt. These feedback loops are difficult to address and can occur gradually over time, especially with models that are updated infrequently. There are two types of feedback loops we should concern ourselves with: 

### 4.1 Direct Feedback Loops

This is usually the most obvious. If a data source is being influenced by the output of a model, there is a clear change in the data sources distribution over time as the model affects the data source. 

**Solution**: 

- Introduce a level of randomization into the model output or isolate certain parts of the data from being influenced by the model.

### 4.2 **Hidden Feedback Loops**:

While direct feedback loops are statistical challenges that ML engineers/researchers can explore, *hidden* feedback loops are another case altogether. Hidden feedback loops are where two systems influence each other indirectly throughout the world. Consider the following example: 
There exist two stock market prediction models from two independent companies. The systems are completely disjoint. Improvements (or worse, bugs!) may cause one to influence the buying and selling behaviour of the other.

- I wish I could add a solution here. Unfortunately, this problem is far beyond the scope of just a single one-liner.

## 5. ML-Systems Anti-Patterns

![Tech Debt](https://matthewmcateer.me/media/ml_tech_debt/ml_tech_debt.png)

The majority of code written for ML-systems is not devoted to learning or even prediction. Often times, the code required may be referred to as *data wrangling*, *cleaning* or according to some, *plumbing*. Below, we discuss several common anti-patterns found in ML code:

### 5.1 Glue Code

A lot of ML engineers tend to use general-purpose solutions for their problems. This often requires *glue code* in which a lot of supporting code is written to get data into and out of these general-purpose packages. This freezes the system to the peculiarities of the general-purpose package. Thus, these packages inhibit future improvements whereby we may want to tweak our objective function or even include domain-specific properties. 

**Solution**: Wrapping these black box packages into common API's allows us to to create more reusable infrastructure and reduces the cost of changing packages in the future.

### 5.2 Pipeline Jungles

Given the amount of data-preparation required for any one single ML project, the number of *pipelines* may grow organically as new signals are identified and new information sources are added. Without care, this can quickly result in a jungle of *scrapes, joins and sampling steps* with intermediate outputs. This makes testing more complicated and costly, introduces a host of maintainability issues and adds cost to further innovation.

**Solutions**:

1. By thinking a little more holistically about data collection and feature extraction, a clean slate approach and redesign, while it may be a major engineering investment, will significantly boost an organizations ongoing costs and speed innovation.
2. Often, glue code and pipeline jungles are symptomatic of an environment where there is a disconnect between those developing the models and those tasked with implementing them. A hybrid research approach where engineers and researchers work on the same team (or even are the same person) can reduce this debt significantly. 

### 5.3 Dead Experimental Code paths

A consequence of the aforementioned anti-patterns is that its easy to create short term experiments to fulfil rapid deadlines for the short term. Initially, this cost is low, but over time, accumulated code paths create increasing debt and backward compatibility and increasing complexity make testing these paths near impossible. 

A famous example is Knight Capital's system losing $465 million in 45 minutes, due to unexpected behaviour from obsolete experimental code paths. 

**Solution**:

- Periodically re-examine each experimental branch and see what can be removed. Often only a small subset of branches are actually used.

### 5.4 Abstraction Debt

Consider the state of abstraction with regards to relational databases. There is a strong, widely accepted set of interfaces. When we consider ML systems, what is the right interface to describe a stream of data, a model or even a prediction? This is a difficult problem that is still a WIP amongst the community. 

For distributed learning the parameter-server abstraction seems to be the most robust approach, however, there are competing specifications of this idea. The lack of standard abstractions can cause the lines to be blurred between components. See this [paper](https://www.cs.cmu.edu/~muli/file/parameter_server_osdi14.pdf) by google researchers for more information on the parameter-server abstraction.

### 5.5 Common Smells

Design smells often indicate an underlying problem in a system. There also exists a set of smells for ML systems:

1. **Plain-old Data Type Smell**: Often, ML systems used and produce rich information. However, they tend to be encoded with plain data types like raw floats and integers. In a robust system, model parameters should know if they are log-odds multiplier or a decision threshold, and predictions should know various pieces of information about how the models produced them and how it should be consumed.
2. **Multiple-Language Smell**: Often, it is tempting to write particular peices of a system in a given language due to the appeal of that languages syntactic sugar, or the convenience of a languages library. However, maintaining and testing in multiple langauges increases cost and difficulty in transferring ownership. 
3. **Prototype Smell**: Small scale prototypes are convenient when testing new ideas. However, regularly relying on such an environment may be an indicator that the full-scale system is brittle, difficult to change or could improve abstractions and interfaces. Maintaining these prototyping environments introduce their own cost and there is danger that time pressures may encourage a prototyping system to be migrating at production. Additionally, results from these prototypes rarely reflect the reality at full scale. 

## 6. Configuration Debt

Any reasonably large software system has a wide range of configurable options. For ML systems we may consider such configurations like, which features are used, how data is selected, learning algorithms, pre and post-processing, verification methods etc. Verification and testing of these configurations may not be seen as important but in a mature system with active development, the number of lines of configuration can exceed the number of lines of configuration. Each configuration has the potential for mistakes. 

Some good rules for configuration systems include:

- Easy to specify a configuration as a small change from a previous version.
- Hard to make manual errors, omissions or oversights.
- Easy to see the difference in configuration between two models.
- Easy to assert and verify basic facts about the configurations eg. Number of features, transitive closure of data dependencies etc.
- Possible to detect unused or redundant settings
- Configurations should undergo full code review and be checking into repositories.

## 7. Dealing with Changes in the External World

The majority of ML systems interact with the external world. Often, the external world is remarkably unstable. The rate of change in the external world creates a system of ongoing maintenance costs with any ML application.

### 7.1 Fixed Thresholds in Dynamic Systems

Often we look to define *decision thresholds* for classification models to perform an action. Often, these thresholds are decided based on trade-offs between metrics i.e. *precision*/*recall*. Setting these thresholds can be manually tasking and have a high associated cost. Especially as data streams evolve over team, our original thresholds may no longer be valid for our goals.

**Solution**:

- One mitigation strategy is to set these thresholds via a simple evaluation on *held out validation data* during online training.

### 7.2 Monitoring and Testing

While traditional e2e tests and unit tests are invaluable, we still need to provide evidence that a system is working as intended. Comprehensive live monitoring of system behaviour in real time combined with automated response is critical for long term system reliability. 

The real question we need to ask our self is: what do we want to monitor? Testable in-variants are not so obvious given that most ML systems are required to adapt over time. Below are some potential starting points:  

1. **Prediction Bias**: Comparing the distribution of predicted labels and ground truth labels is extremely valuable. This is by no means a comprehensive test, as a null model that simply predicts the average values of label occurrences would likely evaluate as a superior model. However, this could be used as a diagnostic for when a model may require attention. Such a monitor would help identify when the training distributions drawn from historical data are no longer reflective of current reality. Slicing prediction bias by various dimensions isolate issues quickly and can be used for automated alerting. 
2. **Action Limits**: If we have a system that automates certain actions (e.g. marking email as spam) we may wish to set a limit on the number of actions as a sanity check. Exceeding this limit for a given action can trigger automated alerts and begin an investigation or manual intervention. 
3. **Up-Stream Producers**: Monitoring the upstream data producers will help to identify any potential downstream affects a ML system may incur. 

In time sensitive environment, real-time responses are crucial. Relying on human intervention in response to alerts is a brittle strategy and therefore creating systems with automated responses is often well worth the investment. 

## 8. Other Areas of ML-related Debt

### 8.1 Data Testing Debt

If data replaces code, it makes sense that this input data should be at the scrutiny of testing as well in a well functioning system. Basic sanity checks are useful and as data gets more sophisticated, so can the monitoring of these input distributions. 

### 8.2 Reproducibility Debt

Scientifically, being able to re-run experiments is crucial for increased reliability. However, in real-world systems, reproducibility is made difficult by randomized algorithms, non-determinism in parallel learning, reliance on initial conditions and interactions with the external world.

There are minor solutions to help mitigate such issues (seeding, consistent weight initialization and frozen data sets) however, none are particularly ideal in production systems. 

### 8.3 Process Management Debt

As systems mature, dozens if not hundreds of models may run simultaneously. This creates a series of problems:

1. How to update configurations of similar models safely?
2. How to assign resources among models with different business priorities?
3. How to visualize and detect blockages w.r.t. data flow?
4. How to develop tools to aid recovery from production incidents?

An important system-level smell we can avoid is common processes with many manual steps. 

### 8.4 Cultural Debt

Rewarding deletion of features, reduction of complexity, improvements in reproducibility, stability and monitoring just as much as accuracy is crucial to a successful ML culture. Such a culture is often found within heterogeneous teams with strengths in both ML research AND engineering. 

## 9. Conclusions: Measuring Debt and Paying it Off

Technical debt is difficult to measure and even harder to predict. Simply due to the fact that debt only becomes apparent over time. While moving quickly through projects may feel like a good indicator of success with regards to debt management, moving quickly may in fact *introduce* debt into a system. Some useful questions to consider are:

1. How easily can an entirely new algorithmic approach be tested at full scale?
2. What is the transitive closure of all data dependencies?
3. How precisely can the impact of a new change to the system be measured?
4. Does improving one model or signal degrade others?
5. How quickly can new members of the team be brought up to speed? 

Paying down ML-related technical debt is cultural shift for many teams. However, this attitude is important for the long term health of successful ML teams!