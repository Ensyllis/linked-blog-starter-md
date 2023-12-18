Directions: There are five exercises below, the five four of which involve identifying a random variable and its  
distribution, then using the PMF to solve the problems. Note that each of the first four problems uses one of the  
following distributions: geometric, negative binomial, hypergeometric, and Poisson. Each distribution is used once.  
For the random variable described in the numbered portion of the first four exercises below, before answering the  
question(s), please do the following:  
1. Define the random variable in the problem using the form X = ::::  
2. Write the random variable in the standard distribution form, with its parameters. Brieáy state why the  
assumptions necessary for X to follow this distribution are satisfied (or approximately satisfied).  
3. Compute the expected value, E(X).  
4. Compute the variance, V (X) :  
5. Sketch a histogram of the PMF. A good sketch simply contains the following properties: the support; the  
approximate mode; the basic shape. It is not necessarily to scale, or pretty; the y-axis need not be labeled.  
Alternatively, you may use R. You should be able to adapt the scripts provided for the example to the exercise  
problems. The functions needed are: dbinom(), dpois(), dgeom(), dnbinom(), dhyper()


These are the current vibes so don't expect well worked out stuff
![[Pasted image 20231026000142.png|400]]

1.
According to exhibits.baseballhallo§ame.org, S.F. Giant Barry Bonds holds the all-time professional single season home run record in 2001 with 73 home runs. (The previous Major League record was Mark McGwire in the Major League at 70 home runs in 1999 and Joe Bauman at 72 home runs in the Minor Leagues in 1954). Suppose that a certain baseball player has a 0.10 probability of hitting a home run. Suppose further that this baseball player has 71 home runs in the season, with three games left with three at-bats per game What is the probability that 

 
X is the number of home runs 
Distribution is Negative Binomial
The random variable is the number of home runs
PMF
$$
\mu = \sum_{x = r}^\infty x{{x - 1}\choose{r-1}}p^r(1-p)^{x-r}
$$
Where r is the number of successes,
Where p is the probability
Where x is the number of trials.

Clearly 
- p = 0.1
- r = 3
- x = 9

E(X):
$$
\frac{3}{0.1} = 30
$$
V(X):
$$
\frac{r \cdot (1-p)}{p^2} = \frac{3 \cdot 0.9}{0.1^2} = 270
$$

![[Pasted image 20231026010241.png]]

(a) The player breaks the record by hitting his 74th home run on the last at-bat of the third game? 

It can be further summarized as the probability of getting 2 targets in the first 8.
Therefore
$$
{8 \choose 2}0.1^2(0.8)^6
$$
We take this and multiply it by 0.1 yielding

0.0149


(b) The player breaks the record by hitting his 74th home run during the season at all? 

$$
{9 \choose 3}0.1^3(0.8)^6
$$
= 0.022020096


2.
Phone calls are received at a certain residence as Poisson distributed with a rate of $\lambda$ = 2 calls per hour. 

Poisson.
X := Number of phone calls received per hour

E(X):
2
V(X):
2

Histogram of the PMF:
![[Pasted image 20231025093410.png]]

(a) If Hulda is trying to get some work done, what is the probability she receives one or fewer calls in an hour? 
0 Calls
```R
> dpois(0,lambda=2)
[1] 0.1353353
```
1 Call
```R
> dpois(1, lambda = 2)
[1] 0.2706706
```
We add em
= 0.4060059

(b) If Hulda takes a 10-min. shower, what is the probability that the phone rings during that time? 

We have to convert the poisson variable to be as such
Y := 2 calls per 60 minutes
which can be simplified to 
Y := $\frac{2}{60} = \frac{x \text{ calls per minute}}{10}$
Multiply both sides by 10
$$
\frac{20}{60} = \frac{1}{3} \text{calls per 10 minutes}
$$
So the probability is 1/3rd

So we want 1- P(0)
P(0) can be found through the following
```R
> dpois(0,lambda=1/3)
[1] 0.7165313
```
Therefore
P(X > 0) = 1 - 0.7165313
= 0.2834687

(c) Using R [dpois() or qpois()]: How long can Huldaís shower be if she wishes the probability of receiving no phone calls to be at least 0.5? 

I just trialed and errored it until I got 
```R
> dpois(0,0.70)
[1] 0.4965853
```
$\lambda = 0.70$
It's close enough for our metrics, 0.69 = lambda is closer but it's a nightmare to work with.
Converting that into hours and minutes we get
So lambda needs to be 0.69 calls per hour converting it once again we get 
$$
\frac{2}{60}
$$
Divide it by 6 so we get
$$
\frac{1/3}{10}
$$
So the probability of getting a call in 10 minutes is 1/3rd.
$$
\frac{1/6}{5}
$$
The probability of getting called is 1/6 per 5 minutes.
Adding enough of it together so the probability of a call is 3/6th per 15 minutes 
Therefore it is at least 50% of no calls, exactly to be precise.
So the answer is 15 minutes.


What I want to do, write a bunch of values for qpois, throw it into the lambda.

3.
Ehud is playing his favorite video game. There is a very difficult part where he only has a 20% chance of passing the level. He has three lives left. Assume that Ehud is sufficiently good at the game that he's not getting better at this part after subsequent attempts (i.e. theyíre independent). What is the probability that 

X:= Geometric 
p = 0.2
n = 3

E(X) :
$$
\frac{1-p}{p}
$$
$$
= \frac{0.8}{0.2}
$$
V(X):
$$
\frac{1-p}{p^2}
$$
$$
= \frac{0.8}{0.2^2} = 20
$$
Histogram:

![[Pasted image 20231025235125.png]]

(a) Ehud passes the level on the third life (attempt)? 
$$
0.8 \cdot 0.8 \cdot 0.2 = 0.128
$$
(b) Ehud passes the level by the third life (attempt)? 
$$
0.2 + 0.8\cdot 0.2 + 0.8^2\cdot 0.2 = 0.488
$$
(c) Ehud takes more than three attempts to pass the level? 
This would require him to fail all three times
$$
0.8^3= 51.2%
$$


4.
The U.S. Forest Service captures, tags, and releases 10 independent deer on a mountain in Yosemite National Park. 

X = number of tagged dears
Hypergeometric:

E(X):
$$
\frac{mn}{N}
$$
We are getting our numbers from a) so 
- m be number of defects (10)
- n be draw size (20)
- N be total size (200)

$$
\frac{200}{200} = 1
$$
V(X):
$$
\frac{10\cdot20(190)(180)}{200^2(199)} = \frac{171}{199}
$$

![[Pasted image 20231026001223.png]]

(a) If the population of deer on the mountain is assumed to be 200 (including the 10 tagged deer), then if the Forest Service captures 20 more deer, independent of one another, what is the probability that at most 1 of them is tagged? 

Let
- N be the population (200)
- K be the success (10)
- n be number of draws (20)
- k be the number of observed successes


For 1 we need to add P(0) and P(1)
$$
P(X = 0) = \frac{{10 \choose 0}{190\choose20}}{200 \choose 20}
$$
And for P(1)
$$
P(X = 1) = \frac{{10 \choose 1}{190\choose 19}}{200 \choose 20}
$$
Adding them we get =0.7372

(b) If the population of deer on the mountain is unknown, then the Forest Service can estimate it in the following way. They sight 20 more deer. Of these 20, 3 are tagged. They use this information to estimate the most likely number of deer in the population. Help the Forest Service by performing the calculations. (Hint: Use the hypergeometric PMF function in R: dhyper() Let the number of untagged deer vary and calculate the probability of getting 3 tagged deer in a sample of size 20 when it is known that 10 tagged deer are out there. Specifically, 
```R
> untagged=25:100 
> > dhyper(x=3 ,m=10, n=untagged, k=20 This approach uses the concept of maximum likelihood.) 
>most_likely_untagged <- untagged[which.max(probabilities)]
> 
> print(most_likely_untagged)
```

56


5.
Do the following 
(a) Using the definition of expected value, show the expected value of the Bernoulli distribution is p: 

Definition: 
$$
\begin{cases}
0, \quad \text{failure}
\\
1, \quad \text{success}
\end{cases}
$$
So we get it such that 
$$
\begin{cases}
0\cdot(1-p), \quad \text{failure}\\
1\cdot p ,\quad \text{success}
\end{cases}
$$
And 
$$
E[X] = 0\cdot(1-p) + 1\cdot p
$$
Anything multiplied by 0 is![[Pasted image 20231026002158.png]]
So clearly 
$$
E[X] = p
$$

(b) Derive the MGF of the Bernoulli distribution. Use the MGF of the Bernoulli distribution to and its mean and variance

$$
M_x(t) = \sum_{x=0}^1P(X=n)e^{tx}
$$
$$
M_x(t)= P(X = 0)e^0+P(X = 1)e^t
$$
$$
M_x(t) = (1-p) + pe^t
$$

Derive it with respect to t
$$
M_x'(t) = pe^t
$$
Letting t = 0 we get
$$
E[X] = p
$$
Now for variance
$$
V(X) = E[X^2]-p^2
$$
$$
M_x''(t) = pe^t
$$
Letting t = 0
$$
p - p^2
$$
can be simplified to 
$$
p(1-p)
$$
Confirming the variance.

