---
layout: post
title: Price/Yield Conversion In Practice
---

When working with Fixed Income trading systems we are often confronted with the requirement to implement a bond security Price/Yield conversion service. 

Many Finance textbooks describe the computation but the provided formulas are too academic and examples are provided only for the special case when the valuation date coincides with the coupon payment date.

This post aims to provide a step-by-step explanation of the Price/Yield conversion used by market practitioners and valid for any valuation date.

We will begin with the process of computing the price, given the yield, of a Federal Republic of Germany debt security maturing on the 4th of January 2020 and paying an annual coupon of 3.25%. 

## Present Value

Let's lay down the pricing parameters first:

| Name                               | Value         |
|------------------------------------|---------------|
| Maturity                           | 01/04/20      |
| Coupon Rate                        | 3.25%         |
| Number of coupon payments per year | 1             |
| Valuation date                     | 04/10/16      |
| Nominal value                      | 100           |
| Day count convention               | Actual/Actual |
| Previous coupon payment date       | 04/01/16      |
| Next coupon payment date           | 04/01/17      |
| Yield                              | 0.2%          |

The next step is to determine the full price of the bond using the following equation:

![1]({{ site.url }}/images/1.jpg)

The full price of the bond is the price including accrued interest.
It is also called "dirty price" or present value.

By convention, the traded price is the “clean price” which can be determined by subtracting the accrued interest
from the “dirty price”. 

It is calculated as follows:

<div class="message">
<em>
P<sup>clean</sup> = P<sup>full</sup> - Accrued Interest
</em>
</div>

## Accrued Interest

Accrued interest equals the coupon times the fraction of the coupon period from the previous coupon payment date to the valuation date.
This fraction is calculated by using the day count convention of the bond which is in our case Actual/Actual.

This gives us:

![2]({{ site.url }}/images/2.jpg)

Which by substituting our values equals:

![3]({{ site.url }}/images/3.jpg)

For regular coupon periods the coupon equation is:

<div class="message">
<em>
Coupon = Coupon rate * Nominal / Number of coupon payments per year = 0.0325 * 100 / 1 = <b>3.25</b>
</em>
</div>

We have now all the values required to compute the Accrued Interest:

<div class="message">
<em>
Accrued Interest = Coupon * Factor = 3.25 * 0.7486338798 = <b>2.4330601093</b>
</em>
</div>

Let's now go back to the equation of the "dirty price":


![1]({{ site.url }}/images/1.jpg)

## Time Periods 

It is worth noting that per ICMA Rule 803.1 the yield is annually compounded.

ICMA Rule 803.1 states:

<div class="message">
<em>
The Association's standard method of calculating maturity yields shall be based on the definition of annual interest compounding. i.e. a bond with a 7% coupon, payable annualy, priced at 100% yields 7% per annum and the same bond paying interests semi-annually yields more.</em>
</div>

The <em>T<sub>1</sub>...T<sub>n</sub></em> time periods are thus expressed in years.


<em>T<sub>1</sub></em> equals to the remaining lifetime of the current coupon:

![4]({{ site.url }}/images/4.jpg)


The subsequent time periods are obtained by adding the year difference between the next coupon and the corresponding coupon to the remaining lifetime of the current coupon.

![5]({{ site.url }}/images/5.jpg)

## Clean Price

The <em>P<sup>full</sup></em> equation thus becomes:

![6]({{ site.url }}/images/6.jpg)

And at the end we obtain the "clean price":

<div class="message">
<em>
P<sup>clean</sup> = 112.3071034523 - 2.4330601093 = <b>109.874043343</b>
</em>
</div>

Which rounded to 5 decimal places is the exact value calculated by the Bloomberg YAS screen.

For convenience, we provide below the computation for each coupon period.

| Date     | Payment | T (Years)    | Discount Factor | Present Value  |
|----------|---------|--------------|-----------------|----------------|
| 04/01/17 | 3.25    | 0.2513661202 | 0.9994978959    | 3.2483681617   |
| 04/01/18 | 3.25    | 1.2513661202 | 0.9975028901    | 3.241884393    |
| 04/01/19 | 3.25    | 2.2513661202 | 0.9955118664    | 3.2354135658   |
| 04/01/20 | 103.25  | 3.2513661202 | 0.9935248168    | 102.5814373317 |
|          |         |              | Sum             | **112.3071034523** |
