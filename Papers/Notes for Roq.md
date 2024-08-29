Here's a digestible summary of the paper you provided on robust query optimization using machine learning (ML) techniques:

### 1. Abstract and Problem Definition
The paper discusses the limitations of traditional query optimizers in relational database management systems (RDBMSs), which often rely on inaccurate parameter estimates and make assumptions that may not hold in practice, leading to suboptimal execution plans. To address these issues, the paper proposes "Robust Optimization of Queries" (Roq), a framework that enables robust query optimization using a risk-aware learning approach. This method aims to improve query performance by incorporating approximate probabilistic ML to predict query execution costs and associated risks more accurately.

### 2. Overall Architecture of the Proposed Method
Roq introduces a novel architecture that integrates a learned cost model with strategies for query plan evaluation and selection based on risk quantification. The architecture comprises:
- **Formalization of robustness** in query optimization.
- **Quantification of uncertainties and risks** associated with different execution plans.
- **Novel algorithms** for evaluating and selecting execution plans based on their predicted cost and risk.

### 3. Experiment Methodology
The experiments are designed to compare Roq's performance against traditional optimization methods and other ML-based approaches. The methodology includes:
- **Generation of synthetic queries** and execution plans using varied scenarios to test robustness.
- **Evaluation of plans** based on their execution time and the accuracy of cost predictions.
- **Use of real-world database systems** like IBM Db2 for realistic benchmarking.

### 4. Results of the Experiments
Roq significantly outperforms traditional and other ML-based query optimization methods in terms of robustness and reliability. It consistently selects more efficient execution plans and shows improved resilience against parameter misestimation and environmental changes.

### 5. Conclusion and Limitations
The paper concludes that Roq offers a practical and effective solution to the challenge of robust query optimization in modern database systems. However, it also acknowledges limitations such as the dependence on the quality of training data for the ML models and the need for periodic retraining to adapt to new data distributions.

# Paper Review
## Introduction
- What query optimization approaches are considered **robust**?
	- they are less sensitive to estimation errors.
	- they do not rely on simplifying assumptions.
- Context: query plan optimization
	- not included query rewrite and query execution
- Limitation of Previous work 
	- Some require intrusive changes in the query execution engine
	- have unreasonable compilation overhead
	- make unrealistic assumptions
	- do not quantify uncertainties
	- do not aim at improving robustness
	- do not use the uncertainties to effectively improve a robustness measure
- a practical approach to solve the robustness problem needs to
	- a) have limited compilation overheads
	- b) avoid making simplifying assumptions
	- c) formalize and quantify the notion of robustness
	- d) quantify the uncertainties associated with the cost estimates
	- e) use the uncertainty estimates effectively to improve a robustness measure.

## Problem Statement —  the problem of robust query optimization

- uncertainty rooted in the
	- plan structure
	-  limitations of the estimation method
	- a crude comparison of a set of plans
- use the terms *risk* and *risky* in this context to point out a high level of uncertainty in each of these dimensions. use the term *robust* when uncertainty is low
- The level of plan robustness is influenced by
	- the structural characteristics
	- the operators used in the plan
	- the error propagation patterns
	- the characteristics of the underlying data
- the sources of uncertainty in the plan evaluation and selection process:
	- Plan risk: the uncertainty inherent to the plan itself. This uncertainty is influenced by the predicates involved, the operators used in the plan, the plan structure, error propagation patterns, and the characteristics of the underlying data.
	- Estimation risk: the uncertainty in cost estimates due to limited knowledge or modeling techniques. This uncertainty is influenced by the limitations in cost modeling such as simplifying assumptions and error-prone parameter estimates.
	- Suboptimality risk: the likelihood of a plan being suboptimal at runtime, after being selected as the optimal one from a set of plans
- **Robust query optimization** necessitates a holistic framework that theoretically defines and quantifies robustness and includes techniques that measure robustness and select plans for execution accordingly. the framework should fulfill the following requirements:
	- Define the notion of robustness and relate this to the 3 types of risks: plan, estimation, and suboptimality risk
	- Provide quantification functions for all 3 types of risks
	- Provide techniques that employ the 3 types of risks in the plan selection task, so that the selected plan is robust
	- Provide techniques for measuring all types of risks
## Theoretical Framework
- uncertainties can be rooted in two sources
	-  the behavior of the alternative query execution plans in the parameter space
	- the limitations of the plan evaluation method (i.e., the cost model)
### Studying and Defining Robustness
#### What Determines Plan Robustness
A robust plan is defined as one that has limited worst-case performance degradation. This property can be characterized by the maximum Suboptimality.
Suboptimality of the plan $p_i$ is the ratio of its execution cost (𝐸𝑇(. )) to the cost of the optimal plan $𝑝_0$ and takes a value in the range of $[1, ∞)$:
$$Subopt(p_i) = \frac{ET(p_i)}{ET(p_o)}$$
The maximum of this ratio over the parameter space of the estimated cost determines the *maximum suboptimality* (**MSO**) and is suggested as a measure of robustness
The plan with smaller MSO is considered more robust.
#### The Impact of Plan Structure on Robustness
- The structure of a plan (e.g., type of join operators) can impact the sensitivity of its execution cost to cardinality estimation errors, and, hence, its robustness.
- the magnitude of cardinality estimation errors grows exponentially with the number of joins.
- Classical cost models are inherently incapable of accounting for the position of operators in the plan in their estimation process. Therefore, in Roq we do not attempt to alter or adapt a classical cost model, instead, Roq proposes a fundamentally different approach using machine learning so that it performs robust query optimization.
#### The Impact of the Cost Model on Robustness
- A robust cost model
	- a) is not sensitive to errors in the input parameter estimates
	- b) does not rely on simplifying  assumptions
	- c) quantifies the estimation uncertainties.
- the ML-based learned models reduce sensitivity to noisy input features which makes them more robust to errors.
- we take a different route than state-of-the-art ML-based query optimizers and design a novel learned  optimizer that estimates risks related to the estimated cost.
### Ensuring Robustness via Risk  Quantification
- Modelling Uncertainty
	- Predictive uncertainty is usually  classified into two types: 
		- data (or aleatoric) uncertainty
			- A neural network can be designed to predict the parameters of the normal distribution, including conditional expected value and conditional variance of the target given the training data and the input sample ($\mathbb{E}[Var(y|x^*,D)]$)
			- Optimizing the following loss function corresponds to maximizing the log-likelihood with a Gaussian prior: $\frac{1}{N}\sum^N_{i=1}{(\frac{\log{\hat{\sigma^2_i}}}{2}+\frac{(y_i-\hat{\mu_i})^2}{2\hat{\sigma^2_i}}+\frac{1}{2}\log{2\pi})}$
				- where $\hat{\mu_i}$ is plugged in from the first branch of the output layer,  $\hat{\sigma_i^2}$from the second one, and $y_𝑖$ from the labels.
		- model (or epistemic) uncertainty is caputured by the captured by the variance of the conditional expected values of the target ($Var(\mathbb{E}[(y|x^*,D))])$)
			- use approximate variational inference by Monte Carlo (MC) Dropout
- Plan Risk
	- the parameter representing the deviation (the variance of the distributions represents the data uncertainty (i.e., the model uncertainty is zero).) of the cost distribution can be considered as the risk inherent to the plan itself.
- Estimation Risk
	- quantify this type of uncertainty employing model uncertainty
	- use as a measure for the Estimation Risk the function in Equation 6
- Suboptimality Risk
	- 𝑅(𝑝𝑋,𝑝𝑌)= 𝑃(𝐶(𝑋)−C(𝑌)>0)
		- Where 𝑅(𝑝𝑋,𝑝𝑌) represents the risk of picking X over Y
- Computation Overheads
- The Independence Assumption
- The Probability Distribution
	- The cost or execution time are not inherently normally distributed variables. However, they can be transformed such that they approximately conform to a normal distribution.
### Risk-aware Plan Evaluation
#### Conservative Plan Selection
rather than picking the plan with minimum  $𝜇_{𝑝_𝑖}$, it picks the plan with minimum $𝜇_{𝑝_𝑖} +𝑓_𝑠· 𝜎_{𝑝_𝑖}$, where $𝑓_𝑠$ is a parameter tuned on a validation set.
#### Plan Selection by Suboptimality Risk
risk minimization problem
#### Search Space Pruning by Plan and Estimation Risks
any plan with either plan or estimation risks above the designated thresholds is eliminated
## Risk-aware Learned Cost Model

Input: query-level and plan-level information
predicts: expected execution time along with its expected variance
- two main modules
	1. learns the representation of query and plan combinations
	2. takes this representation and learns the associations between query-level and plan-level information and their impact on runtime
### Query and Plan Encoding
- Query Encoding
	- use the query’s join graph to encode information about the tables, local predicates, join predicates, aggregations, etc.
	- encode table statistics, such as cardinality and selectivity from local predicates, and correlations between the predicate columns as **node attributes**
	- incorporate statistics about the join predicates, such as join  type, join predicate operator, join column skewness, and join  selectivity, as **edge attributes**.
	- capture high-level characteristics of the join graph, such as number of tables and joins, and the join graph topology, as **global attributes**
- Plan Encoding
	- plan-level information is encoded using a vectorized plan tree that preserves the plan structure
### Representation Learning
- Query Processor
	- use Graph Neural Nets (GNNs) to process the encoded join graph and learn query and table representations.
	- TransformerConv
- Plan Processor
	- takes the vectorized plan tree as input and augments each node with the embedding of the corresponding table(s) produced by the Query Processor.
	- If the plan node is an access operator, it is augmented with the embedding of the corresponding table only.
	- If a node is a subgraph involving multiple tables, the embeddings of the corresponding tables are aggregated using mean-pooling before being concatenated to the plan node.
	- The augmented plan tree is then processed through multiple Tree Convolutional Neural Nets (TCNN) layers, ultimately aggregated into a one-dimensional vector using dynamic pooling
### Estimation Module
- takes the query and plan representations as input
- These two are concatenated into a single vector which is then processed through a MLP. The output of this module is then fed to two separate branches of MLPs.
	- The first branch predicts the expected execution time,
	- The second branch predicts the expected variance (uncertainty) of the execution time.
## Experimental Study
### Experimental Setup
- **Training Data Generation**
	- pairs of queries and plans
	- 13,000 synthetic queries are randomly generated over the TPC-DS dataset
- Plan Generation
	- each query was compiled using 13 different hint sets
- Collecting Labels
	- Each query is executed using each of the guidelines in the pool and the execution time is measured.