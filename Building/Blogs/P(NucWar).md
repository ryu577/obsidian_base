# Modeling the probability of nuclear war

## I) We live in dangerous times
As I write this in November 2022, many people are worried that the world is on the brink of nuclear war. Just look at Google trends for the keyword, "probability". The top related queries people are searching for involve this theme. 

![[google_trends_nuc_war.png]]

Since the internet wants to know, I decided to model this probability with the tools at my disposal. Of course, the reason for this interest is the ongoing war in Ukraine. In this article, we will consider the different states this war can be in. The states can be generalized to any scenario where a big country/ alliance (Goliath) attacks a smaller one (David). These states and the transitions between them can be modeled with a technique called Markov chains. So, let's set out to model the probability of nuclear war with Markov chains and learn about using them in the process. 

In the next section, we'll cover the different states this war can be in and how they can be generalized to other future conflicts of a similar nature. Describing the states will require us to delve a bit into the events of the war. If you're not interested in that part, you can just look at the diagram of states in the next section and skip the rest.

## II) The state space

I saw some threads on Twitter with diagrams showing the various states the war could be in, with some paths leading towards all out nuclear war. Although I couldn't go back and find them, they inspired me to draw out my own:
![[ukr_war_states1.png]]
Let's describe the states in the diagram above:
1) Peace: This is the state we all want. Hostilities cease and the world goes back to some version of "normal". This is the only "green" state and it also happens to be an abosorbing state. By definition, being in this state means we averted a major nuclear exchange.
2) Stalemate: This is the state we are in as of this writing. No side is advancing and the map of the battlefield is almost static. This doesn't mean people aren't dying. The sides continue to lob artillery at each other and attacks happen from time to time, only to be repulsed with high casualty rates.
3) Ukrainian initiative: In this state, Ukraine is advancing. This happened twice recently. First, in  early September when they routed the Russian forces from Kharkiv in the North East. Then, in early November when the Russians withdrew from Kherson city and the surrounding areas North of the Denipro river. Note that in both cases, the momentum was (predictably) brought to an end by rivers that form natural obstacles. One could say that the withdrawal from Kyiv back in April was also the war being in this state.
4) Russian initiative: In this state, Russia is advancing and gaining significant territory. This obviously happened at the very begining of the war. And also when Russian forces seized almost the entirity of the Luhansk oblast (the twin cities of Severodonetsk and Lysychansk being the last to fall).
5) Tactical nuke: This is the state where Russia decides to use a tactical nuke on the battlefield. What saperates a tactical nuke from a "regular" one (which the US dropped on Hiroshima and Nagasaki) is simply the yield, which is a lot lower. Such a strategy would be effective against a large concentration of troops. Since Ukrainian units tend to form in smaller, more agile units it isn't clear what value the use of such a weapon would have. Not to mention a very similar effect can be acheived by just using a lot of conventional artillery (and this wouldn't poison the land or invite a response from NATO, as we will see). Still, Russia might do it just for the shock value.
6) NATO responds in Ukraine: It's dangerous for the world to let Russia use nuclear weapons and face no consequences. If this line is crossed, others (say North Korea or China) will also feel emboldened to use nuclear weapons as their use is normalized. It was important for NATO to deter this. And they told Russian exactly what would happen in nuclear weapons were used, threatening to respond with air power in Ukraine and the Black sea, destroying any Russian military asset they could lay their eyes on (including the Black sea naval fleet). So far, this has been enough to deter the use of tactical nukes. This possibility would become increasingly likely if the Ukrainian offensive were to reach the peninsula of Crimea, which was annexed by Russian back in 2014 and which the Russian public seems to value.
7) NATO responds in Russia: Back when Russia was saber rattling and dropping hints on its use of nuclear weapons, the Polish defense secratary brought up the possibility of a NATO conventional response on Russia itself. Russia has removed a lot of its air defenses around major cities like Vladivostok and repurposed them to ground missiles for use on Ukraine, such is their weapons shortage. In this weakened state, it would be very easy for NATO to target military assets in Russia itself. This remains a remote possibility.
8) Russia nukes Kyiv: Russia might also opt for a more extreme approach and drop a regular nuclear bomb on a major Ukrainian city (like Kyiv) to force them to surrender, just like America did in Japan at the end of the Second world war. 
9) Russia nukes NATO: This is the most extreme step Russia could possibly take. The chances are low, but it could be something the current regime resorts to if they perceived an existential threat.
10) NATO nukes Russia: This is the most extreme step NATO could take and would likely be in response to Russia doing so first.
11) Red peace: Even if a full scale nuclear war does break out between NATO and Russia, it will end at some point. But this peace will be fundamentally different from the state where peace is established without a nuclear exchange. Which is why we separate this from the very first state we described.

As of this writing on the 18th of November 2022, the war is in the state of stalemate. So the question we're going to be asking is, "starting in this state, what is the probability of finally ending up in the "red peace" state (as opposed to the "green peace" state)? 


## III) Continuous time Markov chains 
The state diagram above is a set of states that can transition into each other. A perfect fit for a model called "Markov chains". The system (in this case the war) can be in any of the states. And starting in a given state, there are probabilities associated with every state the system can transition to from there. As an example, going back to our state space, the war is currently in the state of stalemate. We can think of the probabilities of moving to the other states this state is connected to, whenever a transition happens. To take an example, we could go from "stalemate" to "peace" with a probability of 10%, "Russian initiative" (means Russia starts gaining territory) with a probability of 36% and "Ukrainian initiative" (Ukraine starts gaining back its territory) with a probability 54%. 
![[ukr_state_transitions.png]]
We can put all of these probabilities into a transition probabibility matrix where the (i,j)th entry is the probability of transitioning from the i-th state to the j-th state. 

![[ukr_war_transition_prob.png]]

A simplifying assumption in the model is that given you're in the "stalemate" state, the probabilities of transitioning to the other states are what they are. It doesn't matter how we got to the current state. This is called the Markovian property and the reason they're called "Markov chains".  Its a simplifying assumption that says: "the past doesn't matter". The process doesn't hold any grudges or nostalgia. At each instant of time, the current state is all that matters and what past events led up to it are irrelevant. This can also be called the "memory-less" property. This assumption makes the model much more mathematically tractable and any limitations due to the simplicity can be mostly mitigated by making the state space more complicated. 

In real world processes like this war, the transitions to different states happen fluidly in time and not at discrete steps. And for this reason, there is an extension of the discrete step model called continuous time Markov chains. 

Discrete time Markov chains are covered in chapter 4 and continuous time ones in chapter 6 of the book Introduction to probability models by Sheldon Ross. There is a good reason that chapter 5, which covers the exponential distribution, was inserted between them. To extend Markov chains to continuous time, we need a distribution that models the time the process spends in whatever current state it happens to be in. And the Markovian property of memory-lessness should apply also to that continuous distribution. 

*An example of a bus stop*: Say you're at a bus stop waiting for the bus to arrive. The time you have to wait until the bus arrives is given by some continuous distribution. The Markovian property implies that this distribution is unaffected by weather you already waited an hour or weather you just arrived at the bus stop (real bus arrivals don't behave like this since they typically operate on a fixed schedule, so the longer you have waited already, the more the distribution shifts towards smaller additional wait times instead of remaining the same. But motor accidents probably do). 

This seems like a very strict condition on the distribution, one that would severly restrict the pool of candidates we can use. In fact, it restricts the pool to just one. The only distribution that has this property is the exponential distribution. 

And this is why the chapter on the exponential distribution sits between discrete and continuous time Markov chains. The exponential is the only distribution that has the memory-less property. So, the random variables describing the time until transitions can only be exponentially distributed in a continuous time Markov chain. 

Also, given that we are in some state, the random variables defining the next state we're going to transition to and the one describing the time spent in the current state before the transition happens have to be independent random variables. Say this is not the case and the distribution of the time until the next event depends on which state the the transition is going to happen into. In the example above, maybe the transition from stalemate to cease-fire takes about a year, that from stalemate to Russian initiative takes 3 months and stalemate to Ukrainian initiative takes one month. Now, if you knew the war had been in "stalemate" for a 8 months, that would tell you its more likely for the next state to be "peace". And this violates the memory-less property. As time passes, the distributions of the transitions between the states shift.

Now that we've established the properties of continuous time Markov chains, let's define it completely by filling in the parameters so it can be used to answer important questions like the probability of nuclear armageddon.

## IV) Filling in the parameters
We have a few options for parametrizing a continuous time Markov chain. One approach involves building on discrete time Markov chains. 

We could have the transition matrix from the previous section, along with the average times spent in each state (since the distributions of those times are exponential, one parameter is enough to fully define them). 

An alternative is to specify the rates at which transitions between any two states happen. So, instead of the transition matrix in the previous section, we will end up with a rate matrix. We can always get the transition matrix by simply dividing each element by the sum of the row its in (normalizing along the rows). And the rate at which the transition away from the state happens is just the sum of the rates at which transitions to each of the states its connected to happen. This method of parametrization is easier to work with since we don't have to worry about the probabilities of transitions to possible states summing to one. We can look at each transition in isolation.

In the diagram below, I've filled in my estimates of the rates at which every transition happens. For instance, it seems Ukraine gains some kind of initiative every few months (there was the Russian abandonment of the Kyiv campaign, then the liberation of the Kharkiv oblast in early Septermber and most recently the liberation of Kherson). So, I estimated the average time it takes for stalemate to transition to that state to be 90 days. And the rate of that transition to be 1/90. It is more convenient to parametrize in terms of rates instead of average times since we can simply combine rates by adding them, while combining average times is much more tedious.

![[ukr_war_states_with_probs1.png]]

We can also simplify the state space by collecting all states of the same color together into "uber states". This will make answering questions like "what is the probability we will hit any red state" easier to answer. To get the rates of transitions between these "uber states", we can simply sum the rates in the original chain. For example, to get the rate at which the yellow state transitions to the orange state, sum the rates along all the arrows leading from any yellow state to any orange state. Since there is only one arrow going from the yellow states to the green state, the rate associated with the transition from "yellow" to "green" is $\frac{1}{365}$.

![[ukr_war_compr_states.png]]

I put the parameters in [this Excel file][https://github.com/ryu577/stochproc/blob/master/experiments/blg_PNuc.xlsx]. You can download it, play with the parameters and see how the conclusions change. The place where you can enter the rates corresponding to the various transitions is the first sheet labeled "Rates(input)". Once these numbers are changed, everything else updates. We will describe how to look at the results in the next section, where we will also answer our all-important question.

## V) The probability
So, using our model, what is the probability of nuclear war (ever hitting a red state)? The discrete time Markov chain suffices to answer this question since we aren't asking about the time until any event occurs. This is what the transition probability matrix looks like (from sheet labeled "P(Nuclear)" of the Excel file in cell A8):
![[ukr_war_small_chain_orig.png]]

Since the red and green states are absorbing, they never transition to any other state. And the sub-matrix corresponding to just those two states is the identity matrix.

The matrix above defines the transition probabilities for one transition. We are more interested in the long term conclusion, which means what will happen after many iterations of the chain. To get the transition probabilities after two iterations, we simply square the matrix above. Similarly to get the probabilities after $n$ iterations, we multiply the matrix with itself $n$ times. Below is the result of doing this $32$ times (from sheet labeled "P(Nuclear)" of the Excel file in cell G32). 

![[ukr_war_small_chain.png]]

Notice that over time, the probabilities of being in the yellow and orange states go to zero since they are transient states. And the probabilities for the green and red states start summing to $1$ since those are absorbing. And looking at the two numbers to the top-right inside the rectangle, we conclude that starting in the yellow state (where we are as of this writing), the probability of hitting the red state (which involves a nuclear exchange of some sort) is about 8% while the probability of establishing peace without any nuclear exchange is 92%. 

And if we get to the orange state (triggered by Russia using a tactical nuclear weapon), the probabilities of a major nuclear exchange ramps up to 51%.

Note that there is a more efficient way of calculating the probabilities of ending up in the different absorbing states than repeated multiplication of the matrix. It involves a simple closed form solution, see [this post][https://math.stackexchange.com/questions/3037657/state-probabilities-of-a-markov-chain-with-multiple-absorbing-states].

## VI) Other important questions
We can use continuous time Markov chains to answer other questions like: "if a major nuclear exchange is going to happen, how much time do we have on average?". 

Also, the red and green states are not truly absorbing states. The red state will eventually transition back into the green state in a few decades or centuries as the nuclear fallout and all its effects wear out. And the green state is expected to transition back to a yellow state in a few decades when another similar David vs goliath style war is started (it was just a decade ago in 2008 that Russia invaded Georgia). 

Adding those as faint arrows, we get the following state diagram:
![[ukr_war_small_states_complete.png]]

Now this becomes a continuous time Markov chain without absorbing states and it can be used to find the probability that we will be in any of the states at an arbitrary time in the future using the "Kolmogrov forward equations" as covered in theorem 6.2 of the book Introduction to probability models. But that is the subject of a future blog.

#building/blog  #war #history #ukraine #math/prob #math/prob/markov 
created: 221118

[[blogs]]

![[p_nuc_war-min.png#invert]]

![[p_nuc_war_qns.png]]

![[Ukraine war]]
