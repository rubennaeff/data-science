# How Well Can You Predict Student Responses?
_Written by Ruben Naeff_


## NIPS 2014 and Human Propelled Machine Learning

Last month Knewton gathered along with the global data science community at NIPS, which has become one of the top conferences for cutting-edge machine learning. Topics included game theory, climate informatics, energy infrastructure, neural nets and deep learning, and a wealth of different optimization algorithms.

The workshop Human Propelled Machine Learning was of particular interest, as much of the research presented is directly relevant to online learning. Jacob Whitehill (HarvardX) presented a model predicting student dropout in online courses, largely based on log-in times. Jonathan Huang (Google) discussed how to automatically grade programming homework on Coursera by clustering different solutions into equivalence classes. Richard Baraniuk (Rice) introduced the concept of Mathematical Language Processing, analogous to NLP, to interpret mathematical expressions. Nisar Ahmed (CU Boulder) swapped the roles of humans and machines by exploring systems where humans operated merely as a sensor in an automated process, rather than as a controller in operations.

On behalf of Knewton, I presented the paper _On the Limits of Psychometric Testing in Online Education_ (co-authored by Zack Nichols), which provides an in-depth look at the accuracy of several well-known test grading approaches in online education. Traditional grading models were developed for bricks and mortar settings, in which students were tested in one sitting and required to answer all the questions on a test. Today, students may be assessed continuously over the course of a semester, and may not answer all available questions.


## The Naive Way to Predict Student Responses

The most intuitive way to express how proficient a student is at a certain concept is to compute what percentage of questions she answered correctly. If a student S — let’s call her Sarah — attempted ten questions and answered eight correctly, she would get a proficiency score of 80%, which we denote as θS = 80%. The higher the proportion of questions Sarah answered correctly, the more proficient we think she is.

Given this information, how do we predict how Sarah would respond to an arbitrary question? If we don’t know anything about the question, we could argue that Sarah’s proficiency is pretty high, so she probably would give the right answer. Applying this logic to each individual question, we’d predict that Sarah would answer all questions correctly. Analogously, we would predict that her classmate Theo with θT = 45% would answer each question incorrectly.

We call this prediction method the naive classifier, as it naively assumes that a student would respond the same to each question. Obviously, we could do better if we knew a bit more about the questions being answered.


## Classical Test Theory (CTT)

Classical test theory uses an additional parameter, difficulty, to predict student performance. To measure a question’s difficulty, we look at the percentage of students who answered the question incorrectly. For example, if 90% of all students got question Q wrong, then this question is probably very hard. We give Q a difficulty score 90%, or write βQ = 90%.

So, how would Sarah and Theo respond to question Q? Using classical test theory, we simply compare each student’s proficiency with the question’s difficulty. If the student’s proficiency is greater than the question’s difficulty, we predict a correct response. In this example, we predict that both Sarah and Theo will answer Q incorrectly.

One shortcoming of classical test theory is that a student’s proficiency only depends on how many questions a student answered correctly, not which questions she got right. What if Sarah only answered very easy questions? And what if Theo wanted to be challenged and only answered very hard questions? Shouldn’t we assess their proficiency differently? There is a similar problem with how question difficulty is calculated: What if only very knowledgeable students answered a given question? Enter IRT.


## Item Response Theory (IRT)

Like CTT, item response theory uses a parameter for a student’s proficiency and a question’s difficulty, too, but these parameters are computed differently. In short, if a student gets a hard question right, she earns more points than when she gets an easy question right. Mathematically, for the probability P that a student with proficiency θ correctly answers a question with difficulty β, we write formula
Here we compute the θs and βs for all students and questions using Bayesian statistics, which means that we look for the θs and βs that would make the given student responses the most likely to occur.

The formula is illustrated in figure A. This example shows that a student with some proficiency θ (vertical dotted line) is likely to answer Q1 incorrectly (dashed line), but Q2 correctly (solid line).

In our paper, we tested both CTT and IRT on tens of millions of student interactions in two online educational products. We found that CTT is a better model for describing the observed student interactions (i.e., the training set), but that IRT works better for predicting unseen student responses (i.e., the test set). In other words, CTT is more prone to overfitting than IRT.


## Comparing Once Again

Let’s look at the IRT formula once more. If θ < β, then we find that the power e-(θ — β), and that the probability P < ½, which we could round down to zero when making a prediction. Indeed, if a student’s proficiency is less than a question’s difficulty, we’d predict her response to be incorrect. Conversely, if θ ≥ β, then the power e-(θ — β), and hence P ≥ ½, which we would round up to 1. This implies a correct response.

Again, we predict a correct response if and only if a student’s proficiency is greater than a question’s difficulty. When predicting a single response, we are not interested in how much exactly a student’s proficiency or a question’s difficulty is or how far they are apart — we simply need to know which one is bigger.

This means that all we need is an ordering of students and questions, from most proficient to least proficient, and difficult to easy, respectively. A student will correctly respond to all questions below their proficiency, and incorrectly to all questions above it. Rather than learning specific values for θ and β, couldn’t we deduce such an ordering directly from the student responses?


## Bipartite Partial Tournament Ranking

In our paper, we introduce another model for describing student responses. We interpret the student responses as a tournament, in which a student engages in a battle with a question, and either wins or loses. We call this a partial tournament, since students generally do not face all of the questions, and it’s bipartite, since the students only challenge questions and not each other. This tournament is a list of pair-wise comparisons: Which is bigger, the student’s proficiency or the question’s difficulty?

We could draw this tournament as a bipartite directed graph (see fig. C), where vertices represent students and questions, and a student’s outgoing edges represent correct answers, and vice versa. Since we predict a correct response if and only if a student’s proficiency is higher than a question’s difficulty, an edge points to a vertex with a lower parameter. Therefore, paths always go through vertices with decreasing proficiency or difficulty. If we can find a path through all vertices (called a topological sorting), then we will have a ranking of all students and questions that perfectly describes all student responses.


However, such a sorting does not exist if we have cycles (see fig. D). In that case, an edge must be broken (or flipped in the opposite direction) to unambiguously rank students and items. This means that even the most accurate ranking must contain at least one prediction error: The edges you flip are exactly the student responses you will predict inaccurately. Therefore, a model with the highest accuracy tries to find a feedback arc set (FAS): the minimum number of edges that should be removed to eliminate cycles.

It is practically impossible to find the FAS for a large graph, but good approximations exist. In our paper, we combine a well-known algorithm by Eades and a simple local optimization (which we called Eades+LO) which in simulations virtually always found the “true” simulated accuracy.


## Finding the Maximum Accuracy

The above approaches model student behavior with the same total number of parameters (i.e., only one, for proficiency or difficulty), and vary only in their regularizations and fitting algorithms. In our paper, we show that these variations can still result in dramatic differences for in-sample and out-of-sample accuracy (see figures E and F below).


By construction, the tournament ranking-based model will generate a ranking of proficiencies and difficulties that will best describe the given student interactions. Therefore, its in-sample prediction accuracy will always be higher than any other one-parameter model. This is also shown in our paper: On the training set, it outperforms both CTT and IRT by a landslide. On the test set, however, it performs much worse that the other two models.

Even though the tournament ranking performs poorly on the test set, it yields a useful reference point in comparison with other algorithms. For example, if a dataset has a tournament ranking-based prediction accuracy of 85%, then an IRT model fit to this dataset cannot be expected to have a prediction accuracy higher than 85% on neither the training set nor the test set. (As a side note, it’s not difficult to show that this upper bound also holds for 2PL-IRT models.)

So what accuracies can our student predictions maximally reach? Although we ran the model on two very different educational products, the results were remarkably similar. In both products, the tournament ranking-based model reached accuracies between 82% and 100% for predicting all interactions in a concept, averaging around 93%. This suggests that, on average, no one-parameter model will be able to predict the remaining error rate of 7%. This is either noise or student behavior that needs a second parameter to be described.


## Predict How You Will Predict

Surprisingly, these maximum accuracies correlated very well with simple dataset features like mean response score, number of students, and average number of questions answered per student, up to a correlation coefficient R2 of 85%. In other words, without running your tournament ranking or IRT model, you can easily predict how well your model will be able to model student responses.


This is important, since the accuracy of tournament ranking-based predictions is defined by the size of a feedback arc set in the data: Predictability of this feature hints at some conserved student behavior across the datasets. It is possible that modeling approaches which are better tuned to continuous assessment in online educational environments could explain this property.


## Final Thoughts

Classical test theory, which compares the proportion of correct responses, is a very intuitive model that is easy to understand and implement, and performs well in describing student behavior. However, for predicting unseen student interactions, item response theory outperforms the other discussed models. Interpreting the student responses as a tournament of students versus questions and finding the best ranking gives us an upper bound of the prediction accuracy of these models, and all other one-parameter models as well.

The consistency and predictability of maximum accuracies suggest that these models fail to capture hidden patterns in student behavior. One obvious explanation could be that these models ignore the fact that students learn. It stands to reason that an assessment method that can take learning into account should have measurable advantages for this type of data. This work is intended as a step in that direction, by providing a necessary point of comparison.


## Acknowledgements

The paper _On the Limits of Psychometric Testing in Online Education_ was written by Ruben Naeff and Zack Nichols, with valuable input from David Kuntz, Jesse St. Charles, Illya Bomash, Kevin Wilson and Yan Karklin. References can be found in the paper.

