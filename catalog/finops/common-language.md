# Common Language

* `Cost Allocation`: split up the cloud bill and associate the costs to cost centers.
* `Rate Reduction` Cloud providers offer means to reduce rates: reserved instances (RI), committed use discounts (CUD) or commercial agreements between cloud provider an the customer.
* `Wasted Usage`: This encompasses resource usage that is not actively used by the company. Example: An idle VM that someone forgot to remove.
* `Cost Avoidance`: By reducing resource usage or removing a resource altogether or by right sizing it, you drive down cost.
* `Rightsizing` The act of changing the size of provisioned resources to one that fits the needs better. Example: adjust requested CPU for a pod.
* `Cost Savings`: By reducing the rate you pay for resources, you generate savings.
* `On Demand Rate` The normal / base rate of cloud resource usage. Example: the per hour price of a particular VM profile on AWS.
* `Savings Potential`: by looking at the cloud bill forecast, you can predict the amount of savings by applying rate reduction methods (reserved instances etc.)
* `Reservations unused/unutilized/waste`: this describes reserved resources that are underutilized. Also called reservation vacancy. As long as the discount is larger than the cost of the unused reservation, we are still good.
* `Covered Usage`: a resource allocation that is subject to reservation.
* `Coverable Usage`: the ratio of resource usage that is applicable to reservation.
* `CapEx vs OpEx`: 
  * `CapEx`: spending capital makes a resource an asset of the company, which is payed upfront and depreciates over time. You have to right-size it at the beginning and be sure you get value out of it for the years of depreciation.
  * `OpEx`: payed yearly or monthly and accounted when it occures. It is fully deducted in the current tax year.
* `Cost of Capital`:..

Billing Presentation Layer Perspectives:

* `Blended and Unblended Rates`: blended rates apply various kinds of aggregations (e.g. average per unit of storage or per user) on a cost line item. Unblended rates represent the usage cost per resource on the day that it is charged.
* `Amortized Costs`: resource reservation implies upfront cost, e.g. at the beginning of the month. When costs are watched through the lense of "unblended costs", you see a big hit on the first day of the month. An `Amortized Costs` lense evens out the costs over the whole month to a virtual daily line item.
* `Fully Loaded Costs`: this takes an amortized costs view on things, applies discounted rates, factors in shared costs and maps all this to the org structure of the business.
* `Matching Principle`: expenses are mapped to the period of value creation, not to the period of invoicing.
