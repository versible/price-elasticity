# Methodology

## Overview

The core of the methodology is a regression to fit a relationship between unit price and volume.  The complexity lies in the work to assemble "unit price" and "volume" measures such that a regression returns a credible and useful relationship between unit price and volume. 

In the ideal situation, every driver of volume would be known and quantified.  We could then perform a multi-variant regression and determine the relationship between price and volume.  As a byproduct, such a model would also how the relationship for every other driver of volume.  This would be the ultimate grocery retail forecasting model, with value far in excess of price elasticity measurement!  Tempting (and enriching) as achieving this would be, the real world is too complex to permit this.  Every driver of volume is not known, and many are unquantifiable.  We need an approach that copes with incomplete and imperfect data - and to ensure we commnicate results as *our best estimate* with suitably wide bands of certainty (presenting results to 3 decimal places is not a good idea).

Without quantifying every driver of volume, we can use a form of test-versus-control methodology.  
* A store pricing test would be the cleanest implementation of this.  That is, divide stores into 2 representative groups, then move the prices in one group and not the other, then measure the impacts.  This actual pricing experiment is probably a superior approach.  But unrealistic given commercial, customer and operational constraints in a bricks-and-mortar retailer.
* We want to leverage a "natural experiment".  After all, prices move all the time for all maner of reasons.  Our natural experiment involves this week as the test, the same week a year ago as the control.  There are many reasons this is not a perfect test and control definition.  But it is the best available, and while it will produce noisy data, with enough stores and weeks and products we can hopefully see some signal within the noise.

**The crux of the methodology is to calculate a year-on-year price change metric to compare to a year-on-year volume change metric.**

With an appreciation of where the methodology is heading, let's consider it step-by-step.


## Data Preparation
Steps to turn weekly store product level total sales and volume data into a suitable year-on-year price change versus volume change set to run a regression model upon.  This is basically a list of suggested data cleaning, filtering, missing value and outlier removal or replacement steps.

### Stores
Consider total store sales, and count of distinct products sold, as a time-series over the history of the data.  Expect to see a seasonal trend and possibly a long-term trend.  What you don't want to see is steps or gaps in either metric.  These could indicate store closures, refits, major range changes, major roadworks outside the store.  Fortunately, you don't have to investiage or know the cause, just remove problematic looking stores from the analysis.

How to implement code to find a list of problematic stores?  There are many ways:
* Quick and hacky - create a pivotted resultset of stores versus weeks, put in Excel and write a bunch of formula until you have a list of store codes you want to remove.
* SQL based - Normalise each store's sales.  Calculate a smoothed average seasonal trend, then use that to deseasonalise the normalised sales.  Then set some thresholds for when to flag a week as problematic (either absolute threshold, or change versus previous week).
* Python/R based - Use something like an ARIMA time-series model to decompose each store's sales, then a outlier detection library.

### Products
In general, like with stores, you only want to measure products which appear to have pretty stable sales over the history of the data.  So follow a similar approach to stores, to look to product sales as a time-series and flag products where the sales have significant jumps or drops.  (Newly listed products, delists, products changing depth of distribution, etc).

So follow a similar approach to finding problematic looking products as you used for problematic looking stores.  Only maybe tune the thresholds differently.

Excel might creak under the burden of working on product level time-series, given there are typically 1 or 2 orders of magnitude more products than stores.

### Weeks
Weeks can't be delisted, or closed.  But major external factors can have a major impact.  Easter, Christmas, Thanks Giving, Black Friday and all potential causes of problems if they don't line up neatly for year-on-year comparison.  You'll need to think through how to treat each week, the fallback option being to simply exclude these weeks.

Then there are other potential one-off impacts.  Pandemic lockdowns, major sporting events, natural disasters, etc.


### Iterate
Having done a first pass to remove problematic rows of data by store, product and week, build your year-on-year price change and volume change metrics.  Then look at their distribution.  Investigate the big changes.  If you can find common causes?  Will tighten up any of your data preparation logic make the distribution look better?

### Filter based on year-on-year metric values
If year-on-year price change was minor (like less than 2% either way) what use is this data in measuring the relationship between price change and volume.  Kick out these rows of data, they will just be adding noise.  (You could look at the distribution of volume change for these neglible price change rows to see just how much noise there is - it is usual to see a wide distribution, meaning a lot of noise).

Similarily, if year-on-year price changes were massive (like more than 35%) something more than just a price move is probably happening.  Even if it is a BOGOF promotion, that is likely to be accompanied by instore and marketing support, both of which would count as noise if left in the data.  So exclude data rows with very large price changes.

You probably shouldn't exclude data rows based on year-on-year volume changes.  Although there is a line or argument to do so if that argument is considered in isolation.  If a volume has doubled when a price *has not* decreased, something else is clearly at play.  Or if a volume has halved when a price *has* decreased.  Both cases could be removed.  But the problem is we would be trimming our noisy data in a skewed way.  

Imagine you had Normally distributed data centred on +1.0, with a standard deviation of 1.0 then decided to remove data values below 0.  The average of your data would increase to be above +1.0.  You can potentially use year-on-year volume change limits to move the overall price elasticity result.  And hide your rationale behind convincing business reasons.  And almost certainly not get called out by any stakeholders.  Whether this is in your or anyone else's interests is for you to judge.



## Regression
With clean year-on-year price change and year-on-year volume change metrics created, it is time to regress to find the relationship.  There are two ways to approach this:
* Think theoretically about how the nature of the relationship between price and volume is likely to pan out.  It probably isn't linear.  Extremely cheap or extremely expensive prices are likely to have different volume responses (flatter gradients) than more usual prices.  Is this theoretical relationship logarithmic, sigmoid, tanh?  Many happy hours can be spent arguing (probably for very little tangible impact).
* Think practically and plot a bunch of examples on XY scatter plots to get a feel for the distributions.  Typically, this shows very little in the way of correlation.  Any form of best fit line is going to have a very low R-squared.  We really are teasing a weak signal from a lot of noise.  Meaning different forms of regression are likely to have a pretty marginal impact on results.

The best advice is for each modeller to spend some time investigating this topic based on their actual data.  From experience, linear or logarithmic regression usually wins out.  For the non-technical stakeholders, the work at this stage can be the pixie-dust that makes the resultant numbers credible.  It can help to play along with this narative, just don't start believing it yourself!  You are likely teasing a barely believable signal from extremely noisy (seemingly randomly distributed) data.


## Post Regression Adjustments
Statistical purest might be appauled by some of the suggestions here.  Pragmatism is pushed in to conflict with logical rigour, and sometimes logical rigour is sacrifised.  Don't let perfection be the enemy of good enough.  Senior Managers will be making pricing decisions whether informed by a price elasticity model or not.  While we would love to offer a rigourous model, the pragmatic hurdle is offering insight to challenge and improve SME judgment.

There is a school of thought that these adjustments are so egregious they demonstrate how the whole concept of short-term price elastically is fundamentally flawed.  Alternative schools of thought would replace this whole approach with:
* An approach focused on delivering a strategically defined price positions, with assumed costs of deviating from this.
* Advanced simulations built with reinforcement learning that should encompass complex customer behaviour like switching.

Having been suitably warned, here are those post regression adjustments:
* Replace unbelievable elasticities with averages.  
* Overlay broad switching/cannibalisation assumptions.
* Bring overall elasticity numbers in-to-line with previously accepted measurements.


## Presentation

Keep It Simple Stupid.  There is a great temptation, after so much complex work, to want to present a complex set of results.  Resist that temptation!  Spare the complexity only for stakeholders that want to be taken through the full methodology.  
* At most, present a table of elasticities for the 50 or so product groupings.
* Those elasticies should be rounded to 1 or 2 significant figures only (more implies false certainty).
* It might even be preferable to classify product groupings in to 3-5 categories (high, medium, low elasticity), with a range given to each category (low elasticity means we believe the relationship between price and volume is somewhere between 0 and 1...)
* Present the results with key Commercial and Finance stakeholders first.  Get feedback and allow time to address that feedback.  Unveiling results to senior stakeholders at the same time as everyone else is high risk.
* Be humble.  Don't present initial results as gospel truth.  You know that with different data cleaning assumptions, outlier treatment and post-regression adjustments, the numbers can be changed.  Present the data as a model to inform and challenge.  If expert stakeholders disagree with some output, that is their right.  The act of forcing a stakeholder to present their arguments to over-rule model output is adding tremendous value by bringing these arguments in the open.

Simple-Complex-Simple is a superior delivery pattern.
* Simple problem - what effect does price have on volume.
* Complex methodogy - as outlined here.
* Simple output - table with 50 rounded numbers.

*Einstein nailed this pattern with E = mc<sup>2</sup>. Is there a relationship between energy and mass (simple question)? Complex methodology (for anyone that did not do degree level maths).  Simple answer: E = mc<sup>2</sup>.  Clever guy.  Be More Einstein!*




---




Experience points to price elasticity rarely being reliably measured below "product groups" - there is simply too much noise.


