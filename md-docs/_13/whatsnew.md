---
layout: whatsnew
features:
  - title: OpenID Connect
    description: |
      With OpenID Connect now integrated in Couchbase Mobile, you can authenticate users with providers that implement OpenID Connect. This means you won't need to setup an App Server to authenticate users with Google+, PayPal, Yahoo, Active Directory, etc.
    link: 'guides/authentication/openid/index.html'
  - title: TTL
    description: |
      Documents in a local database can now have an expiration time. After that time, they are automatically purged from the database - this completely removes them, freeing the space they occupied.
    link: 'guides/couchbase-lite/native-api/document/index.html#document-expiration-ttl'
sample:
  title: ToDo Sample
  description: |
    This step by steps tutorial walks you through some of the more advanced features of Couchbase Lite and Sync Gateway. Such as running complex map-reduce queries, custom conflict resolution rules, user login and managing all aspects of security in the Sync Function.
  links:
    - name: Start building
      value: 'training/index.html'
---