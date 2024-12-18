# Platforming

## Taxonomy

### McKinsey's Platform Classification 

[source](https://www.mckinsey.com/capabilities/mckinsey-digital/our-insights/the-platform-play-how-to-operate-like-a-tech-company)

* _Customer Journey Platform_: reusable elements impacting customer experience (UI widgets?)
* _Business Capability Platforms_: reusable featues, e.g. payment, identity, inventory management
* _Core IT Platforms_: infrastructure platforms (AWS IaaS), developer platforms on top of infra platforms; shared technology on which journeys and biz capability platforms run; concretely deployment automation, cloud infra and data analytics environments.

### Thoughtworks' Platform Classification

[source](https://www.thoughtworks.com/en-de/insights/blog/platform-tech-strategy-three-layers)

Platforms centralize expertise and decentralize innovation. Commoditization of services, that is open to extension.
There are distinct types, or layers, of Platform which achieve different goals and have different audiences:

1. _Offering/Service Platform_: what most non-technologists mean when talking about Platforms. This means business capabilities platforms like eCom, logistics. Uber's services for drivers seems to match too.
2. _Digital Business Platform_: a programmatic interface (API) to a service platform. Someone writing an alternative mobile client for Uber services would connect here. Such an API can be powerful if it's public and dog-fooding happens from the internal teams. Bezos: "all service interfaces, without exception, must be designed .. to be able to expose the interface to developers in the outside world. No exceptions!".
3. Foundational Technology Platform: treating infrastructure as a sharable platform. Make it easy to use this with deep expertise. 

This is how I understand it, using Amazon as an example:

![thoughtworks-model](https://github.com/user-attachments/assets/37966fb1-586e-499f-a817-9a1132ba0c01)

## Possible Goals of an IT Platform

* Reduce cost: allow orgs to reuse IT assets across a wide range of use cases, reducing cost of new development
* Increase velocity: reuse also accelerates new development and experimentation - you stand on top of a platform. Automated processes and delivery pieplines reduce friction.
* Assure compliance: possibly through curated 3rd party libraries or configuration (limiting access, ensuring audit trails, ..) or through test automation
* Improve transparency: transparency into applications running in the platform, e.g. resource consumption, error rates, patch levels
* Reduced lock-in: platform can act as an abstraction layer over external tools, reducing direct depenencies
* Reliability: we don't want to pay for stuff that's not working; customers don't appreciate bugs and outages. Achivable through k8s, improved alerting (part of transparency)

### Common Blueprint

![common-blueprint](https://github.com/user-attachments/assets/b71e6c92-ee09-4c2c-82e5-7c8bf48d2fb8)

# Platform as a Product, Roadmapping

You can think about product development of platforms in 3 dimensions:

1. Reach: how many people do you want to reach?
2. Breadth: how many platform services do you offer?
3. Depth: per service that you offer, how deep does that go?

Example scenarios:
* High depth, low breadth, medium reach: the platform team offers only K8s, but in depth, supporting it with Helm and automation.
* Low depth, high breadth, high reach: the platform team offers a bunch of services, but it's _plain_ hosting, no support. There is something for everyone, but its very basic.

![Roadmap-Rube](https://github.com/user-attachments/assets/db61017d-2e97-4c0f-ac74-30fafff4c4d1)

## Medidata

* aim to support at least 2 adjacent services in the beginning; as a platform team the service is our product, which has its own lifecycle (release process, updating things, ..). Ideally treat this service lifecycle also as something that we want to automate. The automation should be generic enough that it works for at least the first 2 services. Otherwise we run the risk of having a too tailored approach.

# Implementations

Ready to use pieces:
* <https://www.crossplane.io/registries>
* <https://backstage.io/>

## Anatomy

![Anatomy](./platform_anatomy.jpg)

## Past Org Setups

### eBay

SRE: vault, on call
Platform: k8s, mesh, identity (keycloak)

### Wayfair

Platform: maven poms, buildkite, TFE, Eureka, Backstage, Jazz/Mokka
