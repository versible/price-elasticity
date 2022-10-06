# Price Elasticity - Methodology

## Overview

The core of the methodology is a regression to fit a relationship between unit price and volume.  The complexity lies in the work to assemble "unit price" and "volume" measures such that a regression returns a credible and useful relationship. 

In the ideal situation, every driver of volume would be known and quantified.  We could then perform a multi-variant regression and determine the relationship between price and volume.  As a byproduct, such a model would also show the relationship for every other driver of volume.  This would be the ultimate grocery retail forecasting model, with value far in excess of price elasticity measurement!  Tempting (and enriching) as achieving this ideal would be, the real world is too complex for this to ever be possible.  Every driver of volume is not known, and many are unquantifiable.  We need an approach that copes with incomplete and imperfectly modelled world.

As an sidebar to the above, when we come to publish model output, we must commnicate results as *our best estimate* with suitably wide bands of certainty.  Presenting price elasticity numbers to many significant figures is not a good idea.

Without quantifying every driver of volume, we can use a form of test-versus-control methodology.  
* A store pricing test would be the cleanest implementation of this.  That is, divide stores into 2 representative groups, then move the prices in one group and not the other, then measure the impacts.  This real-world pricing experiment is probably the gold-standard approach.  Unfortunately it is also unrealistic to ever conduct given commercial, customer and operational constraints in a bricks-and-mortar retailer.
* We want to leverage a "natural experiment".  After all, prices move all the time for all maner of reasons.  Our natural experiment involves **this week as the test, the same week a year ago as the control**.  There are many reasons this is not a perfect test and control definition.  But it is the best available, and while it will produce noisy data, with enough stores and weeks and products we can hopefully see some signal within the noise.

**The crux of the methodology is to calculate a year-on-year price change metric to compare to a year-on-year volume change metric.**

With an appreciation of general methodological approach, let's consider it step-by-step.


## Data Preparation
This section outlines steps to consider to turn weekly store product level total sales and volume data into a suitable year-on-year price change versus year-on-year volume change dataset.  What follows is basically a list of suggested data cleaning, filtering, missing value and outlier removal or replacement suggestions.

### Stores
Consider total store sales, and count of distinct products sold, as a time-series over the history of the data.  For any given store, expect to see a seasonal trend and possibly a long-term trend.  What you don't want to see are steps or gaps in either metric.  These could indicate store closures, refits, major range changes, roadworks outside the store, competitor openings nearby...  Trying to investiage, understand and correct for the cause of every step-change in the data is unrealistic.  Just remove problematic looking stores from the analysis.

How to implement code to find a list of problematic stores?  There are many ways:
* Quick and hacky - create a pivotted resultset of stores versus weeks, put in Excel and write a bunch of formula to flag unexpected changes in the trends.  The output should be a list of store codes you want to remove.
* SQL based - Normalise each store's sales.  Calculate a smoothed average seasonal trend, then use that to deseasonalise the normalised sales.  Then set some thresholds for when to flag a week as problematic (either absolute threshold, or change versus previous week).
* Python/R based - Use something like an ARIMA time-series model to decompose each store's sales, then an outlier detection library.

### Products
In general, like with stores, you only want to measure products which appear to have pretty stable sales over the history of the data.  So follow a similar approach to stores, to look at product sales as a time-series and flag products where the sales have significant jumps or drops.  (Newly listed products, delists, products changing depth of distribution, etc).

In terms of implementation, the suggests are akin to those used for stores.  Only with different thresholds for what consitutes an unacceptable "jump" in the metric.


### Weeks
Weeks can't be delisted, or closed.  But major external factors can have a major impact: Easter, Christmas, Thanks Giving, Black Friday...  All such events are potential causes of problems if they don't line up neatly for year-on-year comparison.  You'll need to think through how to treat each week.  The fallback option being to simply exclude these weeks.

Then there are other potential one-off impacts.  Pandemic lockdowns, major sporting events, natural disasters, etc.  These will need to considered and treated individually, with the most probably action being to exclude effected weeks from the model.


### Iterate
Having done a first pass to remove problematic rows of data by store, product and week, build your year-on-year price change and volume change metrics.  Then look at their distribution.  Investigate the big changes.  Can you find common causes?  Does changing your data preparation logic make the distribution look better?  You will probably need to iterate several times to convince yourself you have a good set of data preparation steps.

### Filter based on year-on-year metric values
If a given week's year-on-year price change was minor (like less than 2% either way) what use is this data in measuring the relationship between price change and volume?  The suggestion is to remove these rows of data as they will just be adding noise.  (You could look at the distribution of volume change for these neglible price change rows to see just how much noise there is - it is usual to see a wide distribution, meaning a lot of noise).

Similarily, if year-on-year price changes were massive (like more than 35%) something more than just a simple price move is probably happening.  Even if it is a BOGOF promotion, that is likely to be accompanied by instore and marketing support, both of which would count as noise if left in the data.  So exclude data rows with very large price changes.

You probably shouldn't exclude data rows based on year-on-year volume changes.  Although there is a line of argument to do so.  But this argument runs in to a challenge if you consider the likely impact on the model output.  If a volume has doubled when a price *has not* decreased, some other (unknown) driver is in play.  Or if a volume has halved when a price *has* decreased.  Both cases could be removed.  But the problem is we would be trimming our noisy data in a skewed way.  

Imagine you had Normally distributed data centred on +1.0, with a standard deviation of 1.0.  Then decided to remove data values below 0.  The average of your data would increase to be above +1.0.  This is essentially what excluding some year-on-year volume change rows would do; rig the model to change the overall price elasticity result.

I'm not saying don't do this.  You can present a convincing business reason to do it (removing impacts from unknown drivers of volume).  Most stakeholders will probably never realise this might skew the overall results.  Only you can judge whether this is in your, or anyone else's, interests.



## Regression
With clean year-on-year price change and year-on-year volume change metrics created, it is time to regress to find the relationship.  There are two ways to approach this:
* Think theoretically about how the nature of the relationship between price and volume is likely to pan out.  It probably isn't linear.  Extremely cheap or extremely expensive prices are likely to have different volume responses (flatter gradients) than more usual prices.  Is this theoretical relationship logarithmic, sigmoid, tanh?  Many happy hours can be spent arguing (probably for very little tangible impact).
* Think practically and plot a bunch of examples on XY scatter plots to get a feel for the distributions.  Typically, this shows very little in the way of correlation.  Any form of best fit line is going to have a very low R-squared.  We really are teasing a weak signal from a lot of noise.  Meaning different forms of regression are likely to have a pretty marginal impact on results.

The best advice is for each modeller to spend some time investigating this topic based on their actual data.  From experience, linear or logarithmic regression usually wins out.  For the non-technical stakeholders, the work at this stage can appear to be the magic pixie-dust that makes the resultant numbers credible.  It can help to play along with this narative, just don't start believing it yourself!  You are likely teasing a barely believable signal from extremely noisy data.


## Post Regression Adjustments
Statistical purest might be appauled by some of the suggestions here.  Pragmatism is pushed in to conflict with logical rigour, and sometimes logical rigour is sacrifised.  Don't let perfection be the enemy of good enough.  Senior Managers will be making pricing decisions whether informed by a price elasticity model or not.  While we would love to offer a rigourous model, the more realistic target to set is whether the model is offering insight to challenge and improve SME judgment on price changes.

There is a school of thought that these adjustments are so egregious they demonstrate how this approach to the whole concept of short-term price elastically is fundamentally flawed.  Alternative schools of thought would replace this whole approach with:
* An approach focused on delivering strategically defined price positions, with assumed penalties of deviating from this.
* Advanced simulations built with reinforcement learning that should encompass complex customer behaviour like switching.

Having been suitably warned, here are those post regression adjustments to consider:
* **Replace unbelievable elasticities with averages**.  Elasticity results suggesting volume increases when price increases are not believable (except for high-end luxury goods).  Similarly there is a believability limit to how much volume increase to expect from a price decrease.  Where this believability limits lie is pure judgment.
* **Overlay broad switching/cannibalisation assumptions**.  As noted at the outset, the methodology considers every price/volume relationship in isolation.  But if the price of one jam goes up, and volume declines, chances are not all that volume is lost to the retailers but a proportion switches to another jam product.  Of course, if price elasticity numbers are being used to assess impact on just a single product, switching doesn't matter.  But if the use-case is an overall impact, you might want to overlay some "scaling factors" to reduce the measured price elasticity numbers to reflect likely switching effects.  Measuring switching effects is a whole different modelling challenge.  If you have results to hand, use them.  If not, the simplest approach is probably an SME assessment for each product group as to if switching is likely to be low medium or high (with a judgment call on what low medium and high mean in terms of scaling factors).
* **Bring overall elasticity numbers in-to-line with previously accepted measurements.**  The most egregous suggestion comes last.  To simply scale the whole resultset up or down to mean some accetped overall number, usually measured by some very expensive consultants in the distant past and an immutable part of senior management beliefs.  An excuse to hide behind could be a "scaling factor" to account for medium and long term price-volume effects.  That said, it would be far better to challenge and change senior management beliefs.  But you face the risk that the whole effort is dismissed if you fail.  Good luck which ever way you decide to go!


## Presentation

Keep It Simple Stupid.  There is a great temptation, after so much complex work, to want to present a complex set of results.  Resist that temptation.  Spare the complexity only for stakeholders that want to be taken through the full methodology.  
**At most, present a table of price elasticities for the 50 or so product groupings.**
* Those elasticies should be rounded to 1 or 2 significant figures only (more implies false certainty).
* It might even be preferable to classify product groupings in to 3-5 categories (high, medium, low elasticity), with a range given to each category (low elasticity means we believe the relationship between price and volume is somewhere between 0 and 1...)
* Present the results with key Commercial and Finance stakeholders first.  Get feedback and allow time to address that feedback.  Unveiling results to senior stakeholders at the same time as everyone else is high risk.
* Be humble.  Don't present initial results as gospel truth.  You know that with different data cleaning assumptions, outlier treatment and post-regression adjustments, the numbers can be changed.  Present the data as a model to inform and challenge.  If expert stakeholders disagree with some output, that is their right.  The act of forcing a stakeholder to present their arguments to over-rule model output is adding tremendous value to the decision making process (and your organisation) by bringing these arguments in to the open.

Simple-Complex-Simple is a superior delivery pattern.
* Simple problem - what effect does price have on volume.
* Complex methodogy - as outlined here.
* Simple output - table with 50 rounded numbers.

*Einstein nailed this pattern with E = mc<sup>2</sup>. Is there a relationship between energy and mass (simple question)? Complex methodology (for anyone that did not do degree level maths).  Simple answer: E = mc<sup>2</sup>.  Clever guy.  Be More Einstein!*

---

© Versible 2022