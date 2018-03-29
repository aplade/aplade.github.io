---
layout: post
comments: true
title: Price/Yield Conversion In Practice
---

![PYC]({{ site.url }}/images/money-256294_1920.jpg)
<span class="justify">
When working with Fixed Income trading systems we are often confronted with the requirement to implement a bond security Price/Yield conversion service. Many Finance textbooks describe the computation but the provided formulas are too academic and examples are provided only for the special case when the valuation date coincides with the coupon payment date. This post aims to provide a step-by-step explanation of the Price/Yield conversion used by market practitioners and valid for any valuation date.
</span>

<span class="justify">
We will begin with the process of computing the price, given the yield, of a Federal Republic of Germany debt security maturing on the 4th of January 2020 and paying an annual coupon of 3.25%. 
</span>

## Present Value
Let's lay down the pricing parameters first:

| Name                               | Value         |
|------------------------------------|---------------|
| Maturity                           | 04/01/20      |
| Coupon Rate                        | 3.25%         |
| Coupon frequency                   | 1             |
| Valuation date                     | 04/10/16      |
| Nominal value                      | 100           |
| Day count convention               | Actual/Actual |
| Previous coupon payment date       | 04/01/16      |
| Next coupon payment date           | 04/01/17      |
| Yield                              | 0.2%          |

<span class="justify">
The next step is to determine the full price of the bond using the following equation:
</span>

\\[P^{full} = \\frac{c\_{1}}{(1 + y)^{T\_{1}}} + \\frac{c\_{2}}{(1 + y)^{T\_{2}}}+\\dotsm+\\frac{Nominal\\ value + c\_{n}}{(1 + y)^{T\_{n}}}\\]

<span class="justify">
The full price of the bond is the price including accrued interest. It is also called "dirty price" or present value.
</span>

<span class="justify">
By convention, the traded price is the “clean price” which can be determined by subtracting the accrued interest from the “dirty price”. 
</span>

It is calculated as follows:

\\[P^{clean} = P^{full} - Accrued\\ Interest\\]

## Accrued Interest

<span class="justify">
Accrued interest equals the coupon times the fraction of the coupon period from the previous coupon payment date to the valuation date.
This fraction is calculated by using the day count convention of the bond which is in our case Actual/Actual.
</span>

This gives us:

\\[Factor = \\frac{Days(Previous\\ coupon\\ date,\\ Valuation\\  date)
}{Days(Previous\\ coupon\\ date,\\ Next\\ coupon\\ date)}\\]

Which by substituting our values equals:

\\[Factor = \\frac{Days(04/01/16, 04/10/16)
}{Days(04/01/16, 04/01/17)} = \\frac{274}{366} = \\textbf{0.7486338798}\\]


For regular coupon periods the coupon equation is:

\\[Coupon = \\frac{Coupon\\ rate \\times Nominal
}{Coupon\\ frequency} = \\frac{0.0325 \\times 100}{1} = \\textbf{3.25}\\]

<span class="justify">
At present, we have the values required for the computation of the Accrued Interest:
</span>

\\[Accrued\\ Interest = Coupon \\times Factor = 3.25 \\times 0.7486338798 = \\textbf{2.4330601093}\\]

## Time Periods 

Let's now go back to the equation of the "dirty price":

\\[P^{full} = \\frac{c\_{1}}{(1 + y)^{T\_{1}}} + \\frac{c\_{2}}{(1 + y)^{T\_{2}}}+\\dotsm+\\frac{Nominal\\ value + c\_{n}}{(1 + y)^{T\_{n}}}\\]

<span class="justify">
It is worth noting that yields \\((y)\\) are annually compounded as we use the German Fixed Rate Bonds method in which yields are compounded on the same frequency as the coupon frequency.
</span>

The <em>T<sub>1</sub>...T<sub>n</sub></em> time periods are thus expressed in years.


<em>T<sub>1</sub></em> equals to the remaining lifetime of the current coupon:


\\[T\_{1}  = \\frac{Days(Valuation\\  date,\\ Next\\ coupon\\ date)
}{Days(Previous\\ coupon\\ date,\\ Next\\ coupon\\ date)} = \\frac{92}{366} = \\textbf{0.2513661202}\\]

<span class="justify">
The subsequent time periods are obtained by adding the year difference between the next coupon and the coupon <b><em>c<sub>i</sub></em></b> to the remaining lifetime of the current coupon.
</span>

\\[
\\begin{align}
T\_{2}& = T\_{1} + 1\\\
T\_{3}& = T\_{1} + 2\\\
\\vdots\\\
T\_{n}& = T\_{1} + n - 1
\\end{align}
\\]

## Clean Price

The <em>P<sup>full</sup></em> equation thus becomes:

\\[P^{full} = \\frac{3.25}{(1 + 0.002)^{0.2513661202}} + \\frac{3.25}{(1 + 0.002)^{1.2513661202}}+\\frac{3.25}{(1 + 0.002)^{2.2513661202}}+\\frac{103.25}{(1 + 0.002)^{3.2513661202}}=\\textbf{112.3071034523}\\]

And at the end we obtain the "clean price":

\\[P^{clean} = 112.3071034523 - 2.4330601093 = \\textbf{109.874043343}\\]

<span class="justify">
Which rounded to 5 decimal places is the exact value given by the Bloomberg **YAS** screen, the reference for market operators.
</span>

For convenience, we provide below the computation for each coupon period.

| Date     | Payment | T (Years)    | Discount Factor | Present Value  |
|----------|---------|--------------|-----------------|----------------|
| 04/01/17 | 3.25    | 0.2513661202 | 0.9994978959    | 3.2483681617   |
| 04/01/18 | 3.25    | 1.2513661202 | 0.9975028901    | 3.241884393    |
| 04/01/19 | 3.25    | 2.2513661202 | 0.9955118664    | 3.2354135658   |
| 04/01/20 | 103.25  | 3.2513661202 | 0.9935248168    | 102.5814373317 |
|          |         |              | Sum             | **112.3071034523** |

# Computing the Yield

<span class="justify">
We'll now take the opposite direction and compute the yield given the clean price of **109.874043**.
</span>

<span class="justify">
As the above equation cannot be rearranged to give yield as an algebraic function of price the ***Newton-Raphson*** method is used to solve for y.
</span>

<span class="justify">
The Newton-Raphson iteration procedure will, starting from an initial estimate of the yield \\(y_0\\), iterate using more and more precise estimates of the yield until the difference between the \\(P^{clean}\\) derived from using the yield estimate and the given market price of the bond is less than 0.000001.
</span>

## First iteration

Using the initial seed value formula

\\[y_0 = \\frac{Coupon}{P^{clean}}+\left[\\frac{100}{P^{clean}}\right]^{TTM^{-1}} - 1\\]
where \\(TTM\\) is the time to maturity of the bond, we obtain the initial value

\\[y_0 = \\frac{3.25}{109.874043}+\left[\\frac{100}{109.874043}\right]^{3.2513661202^{-1}} - 1 = \\textbf{0.001033
}\\]

By substituting the initial yield into the price formula we obtain the price

\\[P^{clean} = P^{full} - AI = \\frac{3.25}{(1 + 0.001033)^{0.2513661202}} + \\frac{3.25}{(1 + 0.001033)^{1.2513661202}}+\\frac{3.25}{(1 + 0.001033)^{2.2513661202}}+\\frac{103.25}{(1 + 0.001033)^{3.2513661202}}
-2.4330601093
=\\textbf{110.208333}\\]

<span class="justify">
As this price is greater than the given price 109.874043 the above formula is recalculated with a higher yield.
</span>

## Second iteration

According to Newton-Raphson the \\(y\_{1}\\) is determined as:

\\[y_1 = 0.001033 - \\frac{110.208333-109.874043}{-346.399}= \\textbf{0.001998}\\]

where -346.399 is the derivative of the price equation with respect to yield.

<span class="justify">
It is determined by computing the derivative of the price equation and substituting \\(y\\) with \\(y\_{0}\\).
</span>

\\[f^\prime(y)=\\frac{dP}{dy}= \\frac{-T\_{1}\\times c\_{1}}{(1 + y)^{1+T\_{1}}} + \\frac{-T\_{2}\\times c\_{2}}{(1 + y)^{1+T\_{2}}}+\\dotsm+ \\frac{-T\_{n}\\times (Nominal\\ value+c\_{n})}{(1 + y)^{1+T\_{n}}}\\]

Substituting \\(y\\) with \\(y\_{0}\\) gives:

\\[f^\prime(y\_{0})=\\frac{-0.2513661202\\times 3.25}{(1 + 0.001033)^{1.2513661202}} + \\frac{-1.2513661202\\times 3.25}{(1 + 0.001033)^{2.2513661202}}+\\frac{-2.2513661202\\times 3.25}{(1 + 0.001033)^{3.2513661202}}+ \\frac{-3.2513661202
\\times 103.25}{(1 + 0.001033)^{4.2513661202}
}= \\textbf{-346.399}\\]

<span class="justify">
Our new estimate of the yield is substituted into the price formula and gives us a new clean price of
</span>

\\[P^{clean} = P^{full} - AI = \\frac{3.25}{(1 + 0.001998)^{0.2513661202}} + \\frac{3.25}{(1 + 0.001998)^{1.2513661202}}+\\frac{3.25}{(1 + 0.001998)^{2.2513661202}}+\\frac{103.25}{(1 + 0.001998)^{3.2513661202}}
-2.4330601093
=\\textbf{109.874733}\\]

<span class="justify">
The difference between the obtained price and the market price is \\(109.874733-109.874043=0.00069\\) which is greater than the tolerance 0.000001. A new iteration is thus required.
</span>

## Third iteration

<span class="justify">
As the price of the second iteration is greater than the market price we'll add a small margin to \\(y\_{1}\\).
</span>

\\[y_2 = 0.001998-\\frac{109.874733-109.874043}{-345}= \\textbf{0.002}\\]

The derivative -345 is obtained by substituting \\(y\_{1}\\) in the derivative equation.

\\[f^\prime(y\_{1})=\\textbf{-345}\\]

Using the new yield in the price formula we get 

\\[P^{clean} = P^{full} - AI = \\frac{3.25}{(1 + 0.002)^{0.2513661202}} + \\frac{3.25}{(1 + 0.002)^{1.2513661202}}+\\frac{3.25}{(1 + 0.002)^{2.2513661202}}+\\frac{103.25}{(1 + 0.002)^{3.2513661202}}
-2.4330601093
=\\textbf{109.874043}\\]

<span class="justify">
As the new price is equal to the market price no new iterations are required and the yield of the bond is set to **0.2%**.
</span>