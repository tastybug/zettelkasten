# FinOps Lifecycle 

3 Phases

## Inform

Get visibility and a sense of shared accountability by teams on spend and the why.

Proposed steps:
* __Kickoff__: ask questions, communicate intend
* __Gain Visibility__ into what's happening in your cloud environment
* __Achieve Cost Allocation__ by learning which cost center is truly responsible for which usage before making changes 

At the end of this, summarize your findings and data:

1) Context: build context around your data; which cost centers are there
2) Data: show/visualize cost data, maybe plan for optimizations and efficiency
3) Forecast: we can start making forecasts, budgets on which alerts can be build to detect unexpected increases in spend
4) Budgets: describing current spend and limits

### Achieving Cost Allocation

Various tools:
* folders, projects or other kinds of hierachy structures
* tagging, labeling

Attribution goes towards:
* business units
* costs centers
* NOT: people and teams

#### What about shared resources though

This uses a shared K8S cluster as an example. On GCP this entails a fixed cluster maintenance fee plus per-VM charges.

* One can split this up using some estimate (30% of K8S cluster maintenance cost goes to A, 70% to B)
* Sometimes one can realistically model this: calculate the share of the node used by pod resource ask

## Optimize



## Operate
