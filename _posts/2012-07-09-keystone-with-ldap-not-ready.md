---
layout: post
title: "Keystone with LDAP: Not Ready"
category: sysadmin
tags: [openstack, keystone, ldap]
---
I spent much of this last Friday fighting with [Keystone](http://keystone.openstack.org), attempting to get it to use an LDAP server as its identity store. The documentation for this is spotty, and mostly consists of blog posts. Some of which are badly out of date, as the LDAP support was [broken and refactored](http://adam.younglogic.com/2012/02/openstack-keystone-ldap-redux/) during the Essex cycle. What remains certainly implies that the LDAP identity support should be functional. Sadly, this is not the case.

I'm writing this post mostly so that Google can pick up the error messages I saw and hopefully warn others who also try this. I finally found the bug report for this after about 4-5 hours trying to get it working and assuming that I'd just misconfigured it.

The first brokenness that I encounted was probably not the fault of the OpenStack developers. Ubuntu's package is missing a dependency on python-ldap, which manifests as Keystone failing to start, with "CRITICAL No module named ldap". (I found this via strace, but I think it should also be in keystone.log.)

Once I got it to start, I found that I couldn't list users or roles or tenants, getting either "The action you have requested has not been implemented. (HTTP 501)" or "An unexpected error prevented the server from fulfilling your request. 'Identity' object has no attribute 'get\_tenants' (HTTP 500)". I spent quite a bit of time trying to figure out what I had misconfigured before finally coming across a [bug report](https://bugs.launchpad.net/keystone/+bug/983304), indicating that this is a known issue that will be fixed in Folsom. (And is a 9-line patch.)

So as it turns out, the ability to list anything from the auth server is just not supported if you're using LDAP for your identity store. Since you need to be able to get user and tenant IDs in order to assign roles, this makes the LDAP support that exists in Essex basically unusable. I just wish the docs had stated as much.
