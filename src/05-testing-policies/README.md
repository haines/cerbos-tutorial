# Testing policies

> The policies for this section can be found [on Github](https://github.com/cerbos/tutorial/tree/main/src/05-testing-policies/cerbos).

Cerbos allows you to write [tests for policies](https://docs.cerbos.dev/cerbos/latest/policies/compile.html) and run them as part of the compilation stage to make sure that the policies do exactly what you expect. This saves the manual effort of running example requests over and over to ensure the policy logic is as you expect.

A test suite defines a number of resources and principals and the expected result of actions for any combination of them.

To define a test suite, create a `tests` folder alongside your policy folder. In this folder, any number of tests can be fined as YAML but the file must end with `_test`.

As an example, the `contact` policy states that a `user` can create, read and update a contact, but only an `admin` can delete them - therefore you can create a test suite for this like the below:

```yaml
---
name: ContactTestSuite
description: Tests for verifying the contact resource policy
resources: # the resources to test again
  contact_test:
    kind: "contact"
    attr:
principals: # the different users
  admin:
    id: adminId
    roles:
      - admin
    attr:
  user:
    id: userId
    roles:
      - user
    attr:
tests: 
  - name: Contact CRUD Actions
    input: # the resource from above to test and the actions
      requestId: "test"
      resource: "contact_test"
      actions: ["create", "read", "update", "delete"]
    expected: # the expected output for each principal
      - principal: user
        actions:
          create: EFFECT_ALLOW
          read: EFFECT_ALLOW
          update: EFFECT_ALLOW
          delete: EFFECT_DENY
      - principal: admin
        actions:
          create: EFFECT_ALLOW
          read: EFFECT_ALLOW
          update: EFFECT_ALLOW
          delete: EFFECT_ALLOW
```

With this defined, you can now extend the compile command to also run the tests for example:

```sh
# Using Container
docker run --rm --name cerbos -t \
  -v /tutorial:/tutorial \
  -p 3592:3592 \
  ghcr.io/cerbos/cerbos:latest compile --tests=/tutorial/tests /tutorial/policies

# Using Binary
./cerbos compile --tests=/tutorial/tests /tutorial/policies
```

If everything is as expected the output of the tests should be green:

```sh
Test results
= ContactTestSuite (contact_test.yaml)
== 'Contact Actions' by principal 'user' [OK]
== 'Contact Actions' by principal 'admin' [OK]
```

Full testing documentation can be found [here](https://docs.cerbos.dev/cerbos/latest/policies/compile.html).