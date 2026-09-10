# Lab 07 — VPC Personal Companion Plan

[Study index](../README.md) | [Class lab record](../labs/lab-07-creating-a-vpc.md) | [Cheat sheet](../cheat-sheets/lab-07-creating-a-vpc-cheat-sheet.md)

**Decision: 🟢 Yes — Very High Priority / Very High Learning Value**

**Status: planned independent rebuild only. No personal AWS resources have been provisioned by this documentation task.**

> **Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**
>
> **不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。**

## Why rebuild this lab?

VPC networking is foundational SAA knowledge. The class lab demonstrated that a public IPv4 address, a security group rule, and a running EC2 instance are not enough by themselves. Routing, the Internet Gateway, subnet association, security controls, application bootstrap, and protocol must all align.

The personal rebuild should prove that these pieces can be designed and diagnosed without AWS Academy defaults or pre-created roles.

## Phase 1 — reproduce the core concept from scratch

Plan a small, cost-aware VPC and explicitly create/account for:

| Component | Learning purpose |
| --- | --- |
| Custom VPC | Define the private address space |
| Public subnet | Practice subnet CIDR planning and public routing |
| Private subnet | Contrast routing behavior with the public subnet |
| Internet Gateway | Provide the VPC's internet gateway |
| Public Route Table | Add and explain the default route to IGW |
| Private Route Table | Keep private routing explicit |
| Public test workload | Verify the public path without Academy application dependencies |
| Security Group | Permit only the protocol required for the test |
| IAM role / instance profile if needed | Replace Academy's pre-created role deliberately |
| Bootstrap script | Replace the supplied User Data with an independently understood script |
| Logging/observation | Make troubleshooting evidence available |

Do not assume the Academy `vockey`, `Inventory-App-Role`, lab application package, or classroom permissions exist in a personal account.

## Phase 2 — improve beyond the classroom architecture

After the basic version is understood, compare a more production-oriented design rather than blindly adding services:

```text
Internet
   |
Internet-facing entry point if required
   |
VPC across at least two Availability Zones
   |
   +-- Public subnet AZ-A
   +-- Public subnet AZ-B
   +-- Private subnet AZ-A
   +-- Private subnet AZ-B
```

Evaluate:

- Multi-AZ subnet layout and why subnets belong to one AZ.
- Whether workloads need public IPv4 addresses at all.
- Systems Manager Session Manager for administration instead of public SSH/bastion exposure where practical.
- NAT Gateway versus VPC endpoints versus no outbound internet, based on actual workload needs and cost.
- Load balancer + HTTPS if building a real web entry point.
- VPC Flow Logs for network troubleshooting.
- Scoped Security Groups and whether custom NACLs add value for the exercise.

These are **future design decisions**, not claims about the completed Guided Lab.

## 🧒 Mansion design test

Before building, be able to draw this from memory:

```text
Outside world
     |
  IGW gate
     |
VPC mansion
     |
  road signs
   /      \
front    back
 yard     yard
public   private
```

Then explain:

- CIDR = how much address-land the mansion owns.
- Subnet = a section of the estate.
- Route Table = road signs.
- IGW = outside gate.
- Public IPv4 = public mailing address.
- Security Group = guard beside a resource.
- NACL = guard at a subnet boundary.

**Public address + no road = still unreachable. / 有門牌但沒有路，客人還是到不了。**

## Planned troubleshooting drills

The personal rebuild should intentionally test failures one at a time and then restore the correct setting:

- Remove/replace the public default route and predict the result before testing.
- Use a Security Group that does not permit the application port and distinguish that symptom from a routing failure.
- Verify the effect of public IPv4 assignment separately from subnet routing.
- Test the correct `http://` or `https://` scheme for the configured listener.
- Inspect bootstrap/application logs when EC2 status checks pass but the application is unavailable.
- Use Flow Logs or other evidence where useful instead of guessing.

Do not intentionally expose sensitive services or weaken broad security controls just to manufacture failures.

## Security checkpoints

- Do not publish private keys (`*.pem`, `*.ppk`), credentials, account IDs, ARNs, temporary public addresses, or exact resource IDs.
- Prefer SSM for administration when the architecture supports it; avoid opening SSH to the world.
- Scope inbound application access to what the test requires.
- Use IAM roles instead of long-lived credentials on EC2.
- Use HTTPS/TLS for a real public application; classroom HTTP is only a learning/test configuration.
- Treat route reachability and authorization as separate controls.

## Cost checkpoints

- Check current regional pricing before provisioning; do not assume classroom/free-tier economics.
- Public IPv4 addresses can incur charges.
- NAT Gateway can be disproportionately expensive for a small learning lab; add it only when the learning objective justifies the cost.
- Consider VPC endpoint pricing and data-transfer effects before assuming endpoints are cheaper.
- Set an expected lab lifetime and cleanup time before creating resources.
- Use AWS Budgets/alerts as notification tools, not as a hard spending cap.

## Future success criteria

- [ ] Build the VPC without relying on AWS Academy pre-created networking.
- [ ] Explain every CIDR and verify subnets do not overlap.
- [ ] Demonstrate why the public subnet is public using its route table.
- [ ] Demonstrate the difference between a public IPv4 address and an internet route.
- [ ] Explain IGW attachment versus route-table targeting.
- [ ] Explain Route Table versus Security Group versus NACL.
- [ ] Reach a controlled test application using the intended protocol.
- [ ] Diagnose at least one routing fault and one application/security fault by evidence.
- [ ] Record what was rebuilt that AWS Academy previously supplied.
- [ ] Complete cleanup and verify no unexpected billable resources remain.

## Cleanup plan

The future runbook must delete resources in dependency-aware order and verify the result. At minimum:

- [ ] Terminate test EC2 instances and inspect EBS volumes/snapshots.
- [ ] Release intentionally allocated Elastic IPs and check public IPv4 resources.
- [ ] Delete NAT Gateways if the extension created any; wait for deletion before releasing dependent EIPs.
- [ ] Delete VPC endpoints if created.
- [ ] Remove test load balancers/target groups if created.
- [ ] Remove Flow Logs and review their log destination/retention if created.
- [ ] Delete lab-specific Security Groups after dependent ENIs/resources are gone.
- [ ] Delete custom route tables after subnet associations/dependencies are removed.
- [ ] Delete subnets.
- [ ] Detach/delete the Internet Gateway.
- [ ] Delete lab-specific IAM roles/profiles/policies if no longer needed.
- [ ] Delete the VPC.
- [ ] Check every Region used and review billing/resource inventory after cleanup.

**A stopped instance is not cleanup. A deleted VPC is not proof that every supporting billable resource is gone.**
