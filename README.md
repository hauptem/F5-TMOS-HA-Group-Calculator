# F5 TMOS HA-Group Calculator

A single HTML file that models BIG-IP HA group scoring for trunk-based failover. Enter the trunks, thresholds, weights and active bonus for a device pair and the tool shows the HA score each device computes, which device holds the traffic group, and what each possible link loss would do.

Usage: Open `F5_TMOS_HA_Group_Calculator.html` in a modern browser. The tool runs offline and stores nothing outside the page URL.

<img width="1100" height="936" alt="Image" src="https://github.com/user-attachments/assets/9d2f3374-a4cf-4c60-9b3d-0cbd5905380b" />

## What it models

An HA group on BIG-IP assigns a weight to each trunk and scores it by the fraction of links that are up. The device with the higher total holds the traffic group, and the active device adds a bonus to its own total so that small dips do not cause failover. The rules are simple individually but their interaction is not: a weight of 20 with the default bonus of 10 on a two-link trunk never fails over on one lost link, and a weight of 21 does. The tool exists to make those outcomes visible before the configuration reaches a device.

Scoring follows the BIG-IP documentation for 13.0 and later:

- Trunk contribution is `INT(weight × MIN(up, sufficient) / sufficient)`, with sufficient threshold defaulting to the member count. Reaching the sufficient count yields the full weight.
- A trunk with fewer links up than its minimum threshold contributes 0 and zeroes the device score. A device with score 0 is ineligible and receives no bonus.
- Device score is the sum of trunk contributions plus the active bonus on the device that holds the traffic group.
- The traffic group moves only when the peer score is strictly higher. Equal scores keep the current device.

Trunk limits, weight range (10 to 100) and active bonus range (0 to 100, default 10) match the `sys ha-group` and `net trunk` tmsh references.

## Working with the tool

Each member panel lists its trunks. Members, links up, minimum threshold and sufficient threshold are dropdowns bounded by the member count; weight is 10 to 100; the platform limit in the header caps trunk size at 8, 16 or 32 links. Below the table the score row shows the trunk total, the bonus applied and the resulting HA score, and the banner shows ACTIVE or STANDBY.

Every committed change is treated as a monitor interval: dropdowns on change, the active bonus field when it loses focus (the score row updates while typing, but placement is only evaluated on the committed value). If the change makes the peer score higher the traffic group moves, the failover log records it, and the placement persists until a later change moves it again. Fail to Standby forces a move. Auto failback, with a preferred device and a delay in seconds, returns the group once the preferred device has stayed eligible for the whole delay; a link that recovers for less than the delay is logged as a cancelled failback, with the reason (score below peer, ineligible, setting disabled, preferred device changed). The preferred device corresponds to the first device in the traffic group's HA order. This is the mechanism by which a flapping link does or does not thrash a pair, and the tool reproduces it.

The failover matrix lists, for the device holding the group, the outcome of losing one through all links on each trunk at the current bonus. Above it, three lines give the bonus at or below which every single-link loss fails over, the bonus at or above which none does, and the outcome at the current bonus. When some single-link loss fails over at any bonus (it breaches a minimum threshold, or leaves the device with a raw score of 0, which receives no bonus), the second line says so instead of giving a number.

Config sync (Member 2 follows Member 1) copies trunk configuration from Member 1 as it is edited, leaving links up independent so either side can be degraded. Mirror 1 to 2 is a one-shot copy that also copies links up and the active bonus. Copy link places the whole scenario in the URL fragment; opening the link restores it. Values in the fragment are clamped to the same ranges the controls enforce, and trunk names are escaped, so a link cannot inject markup or crash the page.

## Scope

Trunks only. Pools and VIPRION cluster members use the same arithmetic in an HA group but are not represented. VLAN failsafe, gateway failsafe and the load-aware and ordered-list failover methods are outside the model. The pair is two devices; traffic groups spanning more devices are not modelled.

The auto-failback eligibility test (preferred device eligible with a raw score at least equal to the peer) is an interpretation; F5 documents the delay but not the comparison. Validate against a lab pair before relying on it.

## References

- BIG-IP DSC Administration 14.1, Managing Failover
- Maintaining High Availability Through Resource Monitoring 13.0
- BIG-IP DSC Administration 11.4, Understanding Fast Failover
- tmsh reference, `sys ha-group`
- TMOS Routing Administration, trunk interface limits (K1689, K42303533)

Links to each are in the User Guide dialog of the tool.

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Disclaimer

- This solution is **NOT** officially endorsed, supported, or maintained by F5 Inc.
- F5 Inc. retains all rights to their trademarks, including but not limited to "F5", "BIG-IP", "TMOS", and related marks
- This is an independent, community-developed solution that utilizes F5 products but is not affiliated with F5 Inc.
- For official F5 support and solutions, please contact F5 Inc. directly

**Technical Disclaimer:**
- This software is provided "AS IS" without warranty of any kind
- The authors and contributors are not responsible for any damages or issues that may arise from its use
- Always test thoroughly in non-production environments before deployment
- Backup your F5 configuration before implementing any changes
- Review and understand all code before deploying to production systems

By using this software, you acknowledge that you have read and understood these disclaimers and agree to use this solution at your own risk.
