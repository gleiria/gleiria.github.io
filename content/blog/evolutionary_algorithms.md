+++
title = 'Microsimulations - Part 2'
date = 2024-02-19T18:24:31Z
draft = true
+++

### Optimisation and Genetic Algorithms

---------------------------------------------------

When I first came across evolutionary biology and how life on the planet went from unicellular origanisms to the astonishing biodiversity on our planet that we observe nowadays I got completely fascinated by it. Think about about natural selection - the most important mechanism of evolution - at the genome level. Organisms that are more adapted to their environment (fittest in evolutionary jargon) are more likely to survive and pass on the genes that aided their success. This process causes species to change and diverge over time. Genetic mutations that are beneficial to an individual's survival are passed on through reproduction. This results in a new generation of organisms that are more likely to survive to reproduce.


Natural selection - enhanced reproducion by "fitter" individuals - is the most important mechanism of evolution and mechanisms that produce genetic variety are the fuel (have the potential) for evolution to act upon.  



During that period, I got exposed to Darwinian evolution for the very first time and became completely fascinated by it. and how it took us from unicellular organims to biodiversity that we have nowadays. By that time, I had absolutely no idea about what evolutionary computation was so when the idea of using algorithms inspired by natural selection to solve real world problems is completely mind blowing to me. In this article, I will briefly introduce genetic algorithms and, in case you are new to them, give you some intuition on how this fascinating class evolutionary alogrithms work and can be used to find solutions to real world problems.

# Multi-Objective Optimisation 

Multi-objective optimisation is concerned with finding solutions to a problem with multiple, normally conflicting objectives. If you recal from my previous post (link) our problem was clearly defined: In the context of general population screening, we aimed to optimise for the best screening strategy (one that reduces hospitalisations the most) at the lowest possible cost to a health care system. It is clear that these two objectives conflict with one another; the more we screen the less hospitalisations we will have but the costs will be higher. 


# The search space

Imagine that the following vector represents an hypotetical screening strategy to be adopted by a health care system in a 15 years window.

{{< figure src="/images/vector.png" title="" >}}

Each entry in the vector represents the screening strategy for that particular year. For example, at year 1 no one in the population would would be screened, the same for year two, at year three the 30% individuals at most risk would be screened, at year four no one would be screened again, at year five the 70% individuals at most risk would be screened, at year seven 100% of the population would be screened and no more screening would take place until year 15. Thw search space is massive. For each entry in the vector there are 11 possible strategies. Given that we have a vector of size 15 there are 11^15 possible solutions possible solutions to be implemented. It is simply impractical to exaushtively analyse this search space and evaluate how good each different solution is with respect to our objectives (1) minimise hospitalisations by performing screening and 2) keep costs as low as possible). And here is where the magic happens; we give to the algorithm the task of exploring this massive search space and find the best solutions for us. 


## General workflow

1. Initialization

We ask the algorithm to generate a population of candidate solutions. In our case, a bunch of screening vectors as the one we have above would be randomly generated. We have absolutely no idea of how good these solutions are.

2. Evaluation

Each individual (a solution) in the generated population of solutions will be passed to the objective functions that we have. In our case, each solution will be passed to the objective functions so each screening vector will have costs and number of hospitalisations associated to it. 

3. Mating 

We create an offspring population from our initial population. This process has three steps:

    * Selection: We use information from the evaluation step to select the fittest solutions. The fittest solutions are the ones that do the best job at minimising our two objective functions. Like mother nature does, we want to allow the fittest solutions to reproduce and generate offspring. 

    * Recombination: We take pairs of selected parents and recombine them to create a new individual (a new solution). Using the example of our screening vector, we could for example take the some entries in the vector from one parent and other entries from other parent and combined them to form a new individual. At this stage the generation of an offspring is not completed yet. We have to introduce a mutation in this new individual for the process to be completed.

    * Mutation: The process above recombines features from parents to generate new offspring but does not introduce uniqueness to the new individual. That is exatctely the purpose of the mutation step. It takes some property inherited from the parents and changes it. In our case, a mutation can be as simple changing the value of an entry in the vector say from 0.3 to 0.2. This is critical...

The ouput of mating is a new offspring population. Thus, at this stage we have two populations of the same size; the parent population and the offspring population. 

4. Survival

We merge the two populations (parents + offspring) and decide which solutions are worth keeping and which can be discarded. The output of the survival step in a new parent population that will enter the second generation (iteration) of the algorithm and step one starts again.



Previous post:

Now that we can start to test multiple 'what if' scenarions (i.e screening strategies) we may also integrate multi-objective optimisation techniques into the system. What would happen if we would screen children at different ages? Whould that reduce hospitalisations? What if every child was screened at the age of three, and only those at high genetic risk at later stages? How can we optimise for the most effective screening strategies while incurring the lowest possible costs for health care systems? The search space for potential screening strategies is huge, considering a 15-year timeline simulation where each year could employ 11 different screening strategies, for instance, 0 for screening no one, 1 for screening everyone, and the values from 0.1 to 0.9 representing screening the top 10% to 90%, in increments of 10%, of individuals based on the highest genetic risk scores. This creates a massive search space of possibilities, theoretically expanding to 11^15 possible solutions. In the next post in this series, we will explore how multi-objective optimisation techniques can help us to search for "good enough" solutions at the expense of possibly not finding the absolute best solution. We will delve into a fascinating class of heuristic algorithms inspired by natural evolution and the selection of species, known as genetic algortihms, and explore how these powerful computational methods mimic the process of natural selection to iteratively improve upon solutions to complex problems. Stay tunned! 🚀













