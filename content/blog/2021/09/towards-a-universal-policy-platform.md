---
title: Towards a universal policy platform
authors:
    - Flavio Castelli
    - Rafael Fernández López
date: 2021-09-21
---

Kubewarden is a policy framework for Kubernetes. It can be used to
secure your clusters and to ensure they stay compliant with the
rules your organization establishes over time.

By leveraging the power of WebAssembly, Kubewarden allows policy authors
to write policies using traditional programming languages such as Rust, Go,
AssemblyScript and Swift.

Kubewarden policies, once compiled into WebAssembly modules, are then
distributed using regular OCI registries. This allows Operators to have a
consistent way to securely distribute both container images and policies.

Kubewarden is one of the Open Source projects working to provide a
Policy As Code solution to Kubernetes. Historically, the first project in
this space has been [Open Policy Agent](https://openpolicyagent.org/), also known
as "OPA".

Open Policy Agent policies are written using a purpose-built query language
called [Rego](https://www.openpolicyagent.org/docs/latest/policy-language/).

We don't think there's a right or wrong approach when it comes to writing
policies. Both programming languages (Rust and Go) and query languages (Rego)
have advantages and disadvantages.
We strongly believe policy authors should have the freedom to pick the tool that
best suits their needs.

At the same time, we realize that this flexibility can significantly complicate
the lives of Operation teams. Simplicity is paramount when operating Kubernetes;
having different ways to distribute and enforce policies potentially
overcomplicates things.

The Kubewarden team wants to further expand the freedom of policy authors without
compromising the operation experience we currently provide.

We want to provide a cohesive way to distribute policies, a uniform
way to enforce them and a single platform to host them, regardless of
the language used to write those individual policies.

We want to provide a **Universal Policy Platform**.

## WebAssembly to the Rescue

Rego is a query language created by the Open Policy Agent project. The language
is inspired by [Datalog](https://en.wikipedia.org/wiki/Datalog); its
main purpose is to perform queries on some JSON input data and provide some
output as the response.

The good news is that Rego programs can be built into WebAssembly modules using
the official `opa` command line utility. This is documented in depth inside
of the [official documentation](https://www.openpolicyagent.org/docs/latest/wasm/)
of Open Policy Agent.

While building a Rego program into WebAssembly is simple, running the resulting WebAssembly
module requires more effort. The good news is, we did the hard work!

We wrote a Rust library that can be used to invoke Rego policies compiled into
WebAssembly modules.
This library is called Burrego (a burrito with a Rego filling: obvious, isn't it?! 🌯 🤓)
and can be found [here](https://github.com/kubewarden/policy-evaluator/tree/main/crates/burrego).

The Rego language provides some [built-in functions](https://www.openpolicyagent.org/docs/latest/policy-reference/#built-in-functions)
to help with String operations, regular expressions and many more.
When building a Rego policy into a WebAssembly module, some of these built-in
functions are going to be implemented inside of the Wasm file itself; while others
have to be provided at execution time by the WebAssembly runtime evaluating the module.

The Burrego library implements the built-in functions that are not supported natively
by the WebAssembly modules produced by `opa build`. There are still some built-ins that
Burrego does not yet provide; however, based on the policies we have seen in the open,
the ones we already support should be enough for most Kubernetes users.

[This GitHub issue](https://github.com/kubewarden/policy-evaluator/issues/56)
keeps track of the Rego built-ins we have still to implement. Feel free to
comment over there to prioritize our work.

## One Language, Two Frameworks

Open Policy Agent integrates with Kubernetes using the [kube-mgmt](https://github.com/open-policy-agent/kube-mgmt)
sidecar.
More recently, a new Kubernetes integration gained popularity: [Gatekeeper](https://github.com/open-policy-agent/gatekeeper).

Both OPA and Gatekeeper use Rego to write their policies. However,
the way input parameters and other important information are exposed to the policy
and how the policy has to answer differs between the two.

This leads to the unfortunate situation where a Kubernetes policy written for
the OPA runtime will not work as expected if enforced by Gatekeeper, and vice versa.

We don't like this fragmentation in the "Rego landscape". To address this, we worked to ensure
that Kubewarden can execute both policies built for OPA and Gatekeeper
**without any change needed on existing policies**.

## Authoring a Rego Policy

Rego polices can be built into a single WebAssembly module by using the `opa build`
command. No change is required to already existing Rego policies.

We want these WebAssembly modules to be loaded by Kubewarden transparently,
so the end-users of your policy will not have to deal with the subtle differences
between a "Kubewarden native" policy, an OPA policy or a Gatekeeper one.
The WebAssembly module must be enriched with some Kubewarden-specific metadata
to achieve this portability.

Adding metadata to a WebAssembly module produced by `opa build` works in the
same way as modules produced by Rust and Go. This is done using the
`kwctl annotate` command.

Finally, the annotated file can be pushed to an OCI registry with the usual
`kwctl push` command.

Policy authors can also evaluate their policy locally, before pushing it, via
the `kwctl run` command.

If you are already familiar with Kubewarden, you will notice the developer
workflow stays the same, regardless of the programming language used.

All these steps are described in detail inside of our [documentation](https://docs.kubewarden.io/writing-policies/rego/01-intro.html).

## Enforcing a Rego Policy

Kubewarden policies can be enforced inside of a Kubernetes cluster by defining
a `ClusterAdmissionPolicy` object; this is a Kubernetes Custom Resource
provided by the Kubewarden project.

Operators don't have to bother whether the policy was built using one of Kubewarden
SDKs, OPA or Gatekeeper. The way to express settings and to enforce the policy
is always the same.

## Wrapping Up

We have seen what Rego is and how it relates to Open Policy Agent and
Gatekeeper. We have witnessed how combining WebAssembly with Kubewarden can smooth the
differences between the two of them.

We have also seen that the workflows of policy authors and operation teams
are uniform, regardless of the language used to write the policy.

Developer freedom and operation simplicity are top values for the Kubewarden
project. Thanks to WebAssembly we can work towards making
Kubewarden a universal policy platform.

Now the only thing we can request is to try this out! Run your Rego
policies, whether Open Policy Agent or Gatekeeper targeted, and let us
know what you think!

Also, remember to enjoy Rego and burritos!
