# Security

CIA of Security:
* confidentiality: private data is kept private -> encryption
* integrity: data is not tampered with -> signing
* availablity: is the right party having access -> authentication, authorization

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

Cloud resources are the various services that GCP offers, like buckets, databases, VMs.

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
* metadata

Policies are the combincation of principal 
* Basic Roles apply to projects, e.g. project viewer, project owner

* roles cannot remove permissions granted by another role! You have a union of the permissions.


* rights granted on the org level are passed down to all resources. That's why: higher up more restricted rights!
