# Platforming

## One IT Platform Classification 

* customer journey platform: reusable elements impacting customer experience (UI widgets?)
* business capability platforms: reusable featues, e.g. payment, identity, inventory management
* Core IT platforms: infrastructure platforms (AWS IaaS), developer platforms on top of infra platforms; shared technology on which journeys and biz capability platforms run; concretely deployment automation, cloud infra and data analytics environments.

## Thoughtworks' Platform Classifications

See this [article](https://www.thoughtworks.com/en-de/insights/blog/platform-tech-strategy-three-layers).

* 

## Possible Goals 

* Reduce cost: allow orgs to reuse IT assets across a wide range of use cases, reducing cost of new development
* Increase velocity: reuse also accelerates new development and experimentation - you stand on top of a platform. Automated processes and delivery pieplines reduce friction.
* Assure compliance: possibly through curated 3rd party libraries or configuration (limiting access, ensuring audit trails, ..) or through test automation
* Improve transparency: transparency into applications running in the platform, e.g. resource consumption, error rates, patch levels
* Reduced lock-in: platform can act as an abstraction layer over external tools, reducing direct depenencies
* Reliability: we don't want to pay for stuff that's not working; customers don't appreciate bugs and outages. Achivable through k8s, improved alerting (part of transparency)
