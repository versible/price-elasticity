# Grocery Retail Price Elasticity

## Overview
This project aims to allow like-minded companies and individuals to co-operate in using and developing a basic price elasticity measurement model for repeat-purchase low-value items.  The setting is exemplified by grocery retail, but can no doubt be applied to other settings which share similar product purchasing characteristics.

Please read the license conditions before using any part of this project.  

## Ethos

Until there is source code associated with this project, the question of licensing is probably mute.

The intention is to provide this knowledge on a Copyleft (GPLv2 like) basis.  If you don't know what a Copyleft license is, please educate yourself before cloning / copying this repository.

The intention is to make this project free for anyone to use to measure price elasticity within their own organisation.  "Anyone" could be an employee of a retailer, a researcher or a student.

The intention is to discourage companies or individuals from re-selling this work and gaining payment, partly or wholly, as a consequence.  At least, not without seeking explicit permission first.  In principle, permission will be granted, but in exchange for suitable contribution to the project (financial or otherwise - dependent on scale of commercial gain).

Versible can not claim to have invented this methodology.  Merely to be documenting the culmination of decades of experience working across numerous organisations.  The intention for making this methodology openly available is to enable collaborative improvements between individuals and companies that would otherwise be unable to collaborate due to perceived or real conflicts of interest.  

Decades of experience have also shown the true value from a price elasticity model does not lie in any particular prescribed methodology, but the skills and experience to successfully implement a methodology.

But making a methodology openly available we also hope to faciliate some side-effects:
* To make it harder for software and consulting vendors to hide behind a mask of having a "complex proprietory algorithm" which turns out to be inferior to the one presented here. 
* To equipe inhouse Data / Analytics / Insight / Science teams with deliver credible price elasticity models themselves.
* If a company wants Versible to help implement, we are of course happy to be engaged ;-).


## Data Sources
Ideal source data is a multi-million row table containing 3+ years history of weekly product store level sales and volume data.  But compromises from the ideal are possible, providing you understand the consequences.  

More details are given in the separate `Data Sources.md` file in this repository. 


## Methodology

The core of the methodology is a regression to fit a relationship between unit price and volume.  The complexity lies in the work to assemble "unit price" and "volume" measures such that a regression returns a credible and useful relationship between unit price and volume. 

More details are given in the separate `Methodology.md` file in this repository. 

## Code

Until we are aware of a suitably licensed opennly available data set, we are not providing the source code of an implementation of this methodology.  While at first sight the lack of source code might disappoint, anyone properly investing in the topic will realise it is the methodological description that is far more important.  Every specific implementation requires considerable adaptation to reflect different data source definitions, business contexts, data cleaning, outlier removal, etc.  The idea that anyone could clone some code, point at a data source and gain value from the output is unrealistic.

## Caveats of Approach

This methodology treats purchasing behaviour simplistically.  No consideration is given to switching relationships between products (product A cuts its price so shoppers switch to product A from product B).

Also, companies often what to use price elasticity to justify large-scale price moves (typically price cuts, temporary or otherwise).  The elasticities calculated from a model like this one may be the best avaliable to forecast impacts of such events.  But there are limitations of using these elasticity numbers for the large-scale purpose.  Executive decision makers need to be aware of these limitations (and often are not):

* This approach is measuring individual product impacts, not the impact of co-ordinated price moves.
* This approach can only measure relationships within the context of historic price moves.  If a price move is proposed that is outside anything seen before, extrapolation has to be assumed.
* The approach measures short-term volume impacts only.  I.e. price this week delivered volume this week.  No medium or long term impacts are measured (shoppers noticing / appreciating lower prices and increasing visit frequency or basket size).
* Inflationary price impacts are ignored - unless they are somehow built in to the "change in price" metric.  Please contribute to this project approaches to account for this.
* Market price impacts are ignored.  For example, if the whole market has increased prices +10%, but your company has held prices.  Please contribute to this project approaches to account for this.

   

---

© Versible 2022