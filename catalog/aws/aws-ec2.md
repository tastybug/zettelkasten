# EC2 Elastic Compute Cloud

VMs in the cloud. Images are called AMIs.

## Pricing:

_On Demand_: least commitment, you are charged by time running. Good for experimentation or where the work load is spikey, unpredictable. You pay by the hour.

_Reserved Instances_: you commit to a 1 or 3 year contract. You can pay full or in part upfront to get savings.
Best long term for predictable usage and steady state.
Reserved instances can be shared across accounts and even be sold in the Reserved Instance Marketplace when unused.

_Spot Instances_: provide 90% discount compared to on demand pricing, so biggest savings. Spot instances will be killed by on demand customers. If an instance is killed by AWS, you don't pay for the partial hour.

_Dedicated Host Instances_: for when you are required to avoid multi-tenant infra (due to licensing or regulation), e.g. for some Oracle DB.
_Most expensive_ in general, especially on demand. Savings of 70% with Reserved Instances compared to on demand.
