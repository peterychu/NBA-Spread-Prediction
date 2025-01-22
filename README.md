# NBA-Spread-Prediction
Bayesian Data Analysis Project To Predict NBA Game Spread

# Using a Bayesian Hierarchical Model to predict

# future NBA game spreads and try to generate a

# profit on sports betting

## Peter Chu

## November 2024

## 1 Introduction

Sports betting is a fun ”hobby” that millions of Americans take part in every
year. From placing bets on favorite football teams, to trying to make the perfect
March Madness bracket, billions of dollars are moved. However, in the case of
sports betting, it is often like casino gambling. The majority of players lose
all their money, while few take it all. Knowledge and good understanding of
a sport and its players from prior games can help people make educated bets,
but there is no guaranteed way to make correct predictions. This project aims
to capitalize on said prior information to make more confident bets, specifically
for NBA money line bets. Moneyline bets are defined as simply betting on the
overall winner of the match and payout amounts are defined by odds.

This project will use the 2023-2024 NBA regular season as prior data for the
Bayesian hierarchical model which aims to predict the spread of future 2024-
2025 NBA season games. This is in turn to try and win moneyline bets.
Some limitations of this model is that it will only focus on teamwide statistics.
It will not include any specific player effects nor take into account any roster
shuffles.

This project follow these steps:

1. Use past research/academic papers to create a reasonable model to estimate
total points scored by a team.
2. Use all 2023-2024 regular season NBA games as prior information. We use
only regular games because playoffs involves multiple matches between the same
teams which affects the model negatively. For example, the scoring intensities
and adaptions to opponents can affect model assumptions which aims to predict
regular season games, not playoff games.
3. Update the model with the already played 2024-2025 regular season NBA
games and then use the posterior to predict future matches.


4. Use predictions, actual match results, and sports betting website odds to
calculate money gains and losses. It is important to note that we are not
believing/trying to make the model 100% accurate, but rather>50% accurate.
This is so that it is more accurate than simply tossing a coin and betting.

### 1.1 Moneyline betting explanation

Moneyline bets are defined as bets that simply choose the winner of a game.
However, it is important to note that the team that is expected to win will
often have a lower payout than the underdog team. Thus, even if our spread
predictions are correct, that doesn’t mean it is always the best use of money.
Also, it a hit or miss bet, so you either win your money back plus a certain
payout, or lose everything. To understand the notation of a money line bet, it
is often written as: TEAM 1 : +/-XXX TEAM 2 : -/+ XXX, where the +XXX
indicates the payout for correctly betting$100, while the -XXX indicates the
amount required to bet to win$100 [1][2]. For example, one game that we will
try to predict later is the Los Angeles Lakers vs Oklahoma City Thunder game
played on November 29, 2024. The betting websites had the moneyline odds as
LAL +125 OKC -145. This means that if you were to bet$100 on the Lakers,
the underdogs, and the Lakers win, you would win back your initial$100 plus
an additional$125. If you were to bet on OKC winning, you would need to
correctly bet$145 to win$100. The general formula is as follows [2]:

```
Profit for +XXX Odds =
```
```
Odds
100
```
```
×Bet Amount
```
```
Profit for -XXX Odds =
```
#### 100

```
∥Odds∥
```
```
×Bet Amount
```
## 2 Model creation

When creating this model, it was mainly influenced by the papers ”Bayesian Hi-
erarchical Modelling of Basketball Team Performance: An NBA Regular Season
Case Study” [3] by Paul Attard, David Suda, and Fiona Sammut, and ”Bayesian
hierarchical model for the prediction of football results” [4] by Gianluca Baio
and Marta A. Blangiardo. From these papers it was decided to set up the model
as such:


```
Figure 1: Graph of model
```
The distributions and meaning for each parameter is as follow:

```
FTgj|lgjFT,rgjFT∼Negative Binomial(μ=lgjFT,σ=rgjFT) (1)
2 PTgj|lgj2PT,rgj2PT∼Negative Binomial(μ=lgj2PT,σ=rgj2PT) (2)
3 PTgj|lgj3PT,rgj3PT∼Negative Binomial(μ=lgj3PT,σ=rgj3PT) (3)
TPgj=FTgj+ 2× 2 PTgj+ 3× 3 PTgj (4)
```
Wheregrepresents a single match,jrepresents if the game was a home or away
game (1 for home, 2 for away),lrepresents the probability of a shot type being
made and follows as Uniform(0,1) distribution. This is important as it must
take values between 0 and 1.rrepresents the stopping parameters with respect
to shot type.TPrepresents the total points scored in matchgforj(home or
away) team

rgjM was furthered modeled into components withattManddefMreprinting
the attack (offense) and defense intensities,h(g) anda(g) representing the home
and away team index,cMrepresenting the intercept, andhomeMrepresenting
the home advantage. M represents the type of shot being Free Throw (FT), 2
Point Shot (2PT), or 3 Point Shot (3PT).rgjmwas modeled as such:

log(rg (^0) M) =atth(g)M+defa(g)M+cM+homeM (5)
log(rg (^1) M) =atta(g)M+defh(g)M+cM (6)
For each parameter, the prior distributions were chosen:


```
atttM∼Normal(μ=μatt,τ=τatt) (7)
deftM∼Normal(μ=μdef,τ=τdef) (8)
μattM∼Normal(μ= 0,τ= 0.0001) (9)
μdefM∼Normal(μ= 0,τ= 0.0001) (10)
τattM∼Gamma(μ= 0. 1 ,σ= 0.1) (11)
τdefM∼Gamma(μ= 0. 1 ,σ= 0.1) (12)
homeM∼Normal(μ= 0,τ= 0.001) (13)
cM∼Normal(μ= 0,τ= 0.001) (14)
```
Wheretrepresents the team index.

A constraint added for identifiability purposes was the sum-to-zero constraint
and was done by subtracting the mean from eachatttanddeftvalues [4].(There
are 30 teams in the NBA) Thus:

```
Σ^30 t=0atttM= 0 and Σ^30 t=0deftM= 0
```
With the model setup, we can now use our 2023-2024 NBA regular season data
in the model

## 3 2023-2024 NBA regular season data and model

The 2023-2024 NBA regular season data was obtained using the ’nbaapi’ pack-
age [5]. It contains detailed statistics for almost every NBA and G league game
and also includes player statistics.

Using this package, the data was obtained and then transformed to only include
necessary columns which were: the unique game ID, unique team ID, home team
indicator, number of shots made for each shot type, total points, each shot type
percentage made, and the overall game +/-.

```
Figure 2: Sample rows from transformed data
```
However, this data clearly contains 2 rows per game. One for the home team
statistics, and one for the away team statistics. Thus this data needs to be
”unstacked” for our model purposes to avoid double counting statistics.
Now that the data is fully transformed, we can use it in our model. It was run
using 2,000 samples with 1,000 burn ins.


```
Figure 3: Sample rows from transformed and unstacked data
```
```
Figure 4: Sample rows from trace
```
The model ended up having 251 rows where ˆr > 1 .01 and requires investigation.
Due to the large amount of variables and number of samples for each variable, it
is not possible to concisely show it all in figures. Thus, all the variable outputs
are in separate excel files. Noticeably, most variables that had a ˆr > 1 .01 were in
the range of [1.01,1.06], and those that didn’t were primarily the unconstrained
μatt/defThus, we can proceed without worry that these will largely affect the
model.

```
(a) Unconstrained attft (b) Constrained attft
```
```
Figure 5: Unconstrained vs Constrained ˆrcomparison
```
Again to avoid taking up many pages with graphs, all the posterior graphs are
in a separate file.

Some immediate issues that are noted have to due withl 2 ptandl 3 pt. They both
have extremely high mean values and asymmetrical/jagged graphs as seen in
Figure 4.


```
Figure 6: 2023-2024lgraphs
```
The free throw graph for both home and away is fine makes sense, but the graphs
for 2PT and 3PT does not. Since we are following the model from papers [1]
and [2], we will not change it, but for the 2024-2025 model we will change it to
a different distribution that still only produced values in between 0 and 1.

Now that we have parameter estimates for each variables, we can use our 2023-
2024 posterior estimates as our prior estimates for our 2024-2025 model and
then use it to generate predictions for future games.

## 4 2024-2025 data and updating 2023-2024 model

The 2024-2025 NBA regular season data was obtained the same way as the 2023-
2024 NBA regular season data using the ’nbaapi’ package again and performing
the same transformations.

```
Figure 7: Sample rows from transformed data
```
```
Figure 8: Sample rows from transformed and unstacked data
```

Now with our new data set as observed, we can run the model again drawing
2,000 samples with 1,000 burn ins. This time we will changelto follow a
Beta(α= 1,β= 1) distribution. This still only has values between 0 and 1 and
has a mean of 0.5. This will hopefully align ourlvalues to be more realistic.

```
Figure 9: Sample rows from 2024-2025 trace
```
As we can see, the only ˆrvalues greater than 1.01 are still the unconstrainedatt
anddefvariables. Thus our model has converged well, but is still not perfect.
The modellvalues and graphs are still unrealistic. Thus, for future models, this
should be looked into more thoroughly to resolve. For now, we will continue to
use this model for predictions.

```
Figure 10: 2024-2025 modellgraphs
```

## 5 Predictions for selected matches and results

Using our model we can look at future NBA games/matchups and get the pre-
dicted total points for each team. It is important to remember that we did not
model any sort of player effects and only did a team wide model. Thus, this
model does not account for any roster changes made prior to the start of the
season. To see how our model performs, we will get the predicted spread of 3
games, and compare them with actual results. I chose these 3 games played on
November 29, 2024:

1. Los Angeles Lakers (LAL) vs Oklahoma City Thunder (OKC) with LAL
being the home team
2. Miami Heat (MIA) vs Toronto Raptors (TOR) with with MIA being the
home team
3. Chicago Bulls (CHI) vs Boston Celtics (BOS) with CHI being the home team

I will also include the moneyline odds proposed by a online sports betting in-
formation website [6][7][8] to see if our model can generate profit with the as-
sumption of$100 bets. The odds were set as:

1. LAL +125 OKC -
2. MIA -340 TOR +
3. CHI +510 BOS -

Getting the mean of our posterior samples for the proposed matches we have
that:

1. For the LAL vs OKC game our model predicted a +9.28 point lead LAL win,
with the actual results being 93-101 OKC win
2. For the MIA vs TOR game our model predicted a +14.9 MIA win, with the
actual result being 121-111 MIA win
3. For the CHI vs BOS game our model predicted a +4.2 BOS win, with the
actual result being 129-138 BOS win

Thus our model correctly predicted 2 out of 3 games correctly with an expected
profit of: -$100 for the LAL-OKC game,$29.4 for the MIA-TOR game, and
$13.51 for the CHI-BOS game, totaling a total net profit of -$57.07.

## 6 conclusion

Overall we can see that our model had issues, but was still able to generate
predictions. However, this model is not very refined, nor complex enough to
capture all of the randomness in sports nor generate any completely meaningful
predictions. Even though the model correctly predicted 2 out of 3 games, it did
not take into account every factor, and only made the predictions off limited


prior data. Additionally, assuming we had bet$100 on each game, we would be
at a net negative profit. This goes to show that people can not just blindly bet
with even>50% accurate models as the losses exponentially outpace the gains.

I believe that this model had a good setup and was able to generate partially
meaningful predictions and its utilization of a Bayesian hierarchical model led
results that were better than just simply flipping a coin.

I believe that a far, far more complex model could more confidently make predic-
tions, but not necessarily increase the number of accurate predictions. Sports
is noisy, upsets happen, and there is a huge amount of randomness. Thus, I
believe the model created here serves a good base to continue building on top
of.

## 7 References

[1] Sohail, Shehryar. “Sports Betting Odds: How They Work and How to Read
Them.” Investopedia, Investopedia, [http://www.investopedia.com/articles/investing/042115/betting-](http://www.investopedia.com/articles/investing/042115/betting-)
basics-fractional-decimal-american-moneyline-odds.asp. Accessed 1 Dec. 2024.

[2] How to Calculate Potential Payouts in Betting, [http://www.legalsportsreport.com/how-](http://www.legalsportsreport.com/how-)
to-bet/payouts/#: :text=The math behind calculating payouts on sports bets&text=When
the odds are negative,Odds/100 * Stake = Profit. Accessed 2 Dec. 2024.

[3] Attard, Paul, et al. “Bayesian hierarchical modelling of Basketball Team
Performance: An NBA regular season case study.” Proceedings of the 11th In-
ternational Conference on Sport Sciences Research and Technology Support,
2023, pp. 101–111, https://doi.org/10.5220/0012159100003587.

[4] Baio, Gianluca, and Marta Blangiardo. “Bayesian hierarchical model for the
prediction of football results.” Journal of Applied Statistics, vol. 37, no. 2, 21
Jan. 2010, pp. 253–264, https://doi.org/10.1080/02664760802684177.

[5] Swar. “Swar/NBAAPI: An API Client Package to Access the Apis for
Nba.Com.” GitHub, github.com/swar/nbaapi. Accessed 1 Dec. 2024.

[6] https://sportsdata.usatoday.com/basketball/nba/odds/

[7] https://sportsdata.usatoday.com/basketball/nba/odds/

[8] https://sportsdata.usatoday.com/basketball/nba/odds/


