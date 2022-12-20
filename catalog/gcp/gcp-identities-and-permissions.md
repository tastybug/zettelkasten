# Security

CIA of Security:
* confidentiality: private data is kept private -> encryption
* integrity: data is not tampered with -> signing
* availablity: is the right party having access -> authentication, authorization

# Cloud Identity, the Google Identity Provider

Cloud Identity is a GCP product that you can use as an SSO identity provider outside of GCP. When used in GCP, it helps with authentication of members and provides principals for which you then "do your IAM on".

It is possible to sync external user directories to Cloud Identity. This allows (for example) Wayfair employees to log in to GCP with their Wayfair credentials and then do whatever IAM deems fit.

# IAM

Identity and access management deals with the question of who is allowed to do what.
You assign principals to roles, which gives you a policy.

## Principals

IAM supports a variety of principals and means of identity management:
* google account (me@google.com)
* google group (which is Google's mailing list implementation)
* a GCP Cloud Identity (e.g. wayfair.com) tenant (similar to wayfair as a tenant of Okta)
* Google Workspace (fka G Suite), which technically is again a tenant in Cloud Identity
* Service Accounts (i.e. technical users), which are the only accounts that solely exist in GCP

## Permissions

A permission describes what can be done on a particular resource. It's identified by a string like `storage.objects.get`.

## Resources

Cloud resources are the various GCP services (Cloud Storage, Cloud SQL) and INSTANCES of those services.
Examples:
* a concrete bucket
* a concrete object in a bucket
* all buckets in a project
* all VMs in a project

## Roles

Roles are a set of privileges, identified my a string: 
```
roles/storage.objectViewer:
  - resourcemanager.projects.get
  - resourcemanager.projects.list
  - storage.objects.get
  - storage.objects.list
```

There are 3 categories of roles:

1) _Basic Roles_ (project level): predate IAM and are very broad: `Owner` (`roles/owner`), `Editor` (`roles/editor`) and `Viewer` (`roles/viewer`). They affect many (but not all!) resource types. So being an `Editor` mean you are a bucket editor just as you are a database editor.
2) _Predefined Roles_ (resource level): these came with the introduction of IAM and bundle a set of permissions of 1 resource. Example:`roles/storage.objectViewer` being able to list and get content in a bucket.
3) _Custom Roles_ (resource level): allow you to bundle up permissions under a new name, allow fine tuned role shapes.

## Policies

Policies bring the concepts together and form the basis of IAM. You have 3 ingredients:

* the principal
* the role(s)
* optional metadata to make it more fine grained: 
  * time constraints: only until X, only on weekdays?
  * is it the service itself, like Cloud Storage?
  * Is it only concrete buckets, certain VMs with certain tags?
  * and so on

### Deny Policies

It is possible to set up policies that DENY a set of permission. They work just like ALLOW permissions, but are evaluated first.

### How Policies are Evaluated

ALLOW and DENY policies are a union of everything from org down to project level. A more permissive ALLOW policy "beats" the stricter permission. Means: if you're a viewer and an editor, you'll effectively be an editor.

1) IAM goes over all DENY policies. If it finds a match, the request is denied.
2) It then goes over all ALLOW policies. If it finds a match, the request is approved.
