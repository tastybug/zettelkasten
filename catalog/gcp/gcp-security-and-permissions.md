# Security

CIA of Security:
* confidentiality: private data is kept private -> encryption
* integrity: data is not tampered with -> signing
* availablity: is the right party having access -> authentication, authorization

# IAM

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

1) _Basic Roles_: predate IAM and are very broad: `Owner` (`roles/owner`), `Editor` (`roles/editor`) and `Viewer` (`roles/viewer`). They affect many (but not all!) resource types. So being an `Editor` mean you are a bucket editor just as you are a database editor.
2) _Predefined Roles_: these came with the introduction of IAM and bundle a set of permissions of 1 resource. Example:`roles/storage.objectViewer` being able to list and get content in a bucket.
3) _Custom Roles_

## Policies
Policies are the combincation of principal 
* Basic Roles apply to projects, e.g. project viewer, project owner
* Predefined Roles relate to services: app engine admin, storage object viewer
* roles cannot remove permissions granted by another role! You have a union of the permissions.
* custom roles are possible

* rights granted on the org level are passed down to all resources. That's why: higher up more restricted rights!
