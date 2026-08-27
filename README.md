# F5 TMOS HA-Group Calculator

A single HTML file that models BIG-IP HA group scoring for trunk-based failover. Enter the trunks, thresholds, weights and active bonus for a device pair and the tool shows the HA score each device computes, which device holds the traffic group, and what each possible link loss would do.

Usage: Open `F5_TMOS_HA_Group_Calculator.html` in a modern browser. 

<img width="1063" height="914" alt="Image" src="https://github.com/user-attachments/assets/2c6c5b7d-92ef-4d3e-88f3-8d7e91a55e51" />

## What it models

Scoring follows the BIG-IP documentation for 13.0 and later:

- Trunk contribution is `INT(weight × MIN(up, sufficient) / sufficient)`, with sufficient threshold defaulting to the member count. Reaching the sufficient count yields the full weight.
- A trunk with fewer links up than its minimum threshold contributes 0 and zeroes the device score. A device with score 0 is ineligible and receives no bonus.
- Device score is the sum of trunk contributions plus the active bonus on the device that holds the traffic group.
- The traffic group moves only when the peer score is strictly higher.

Trunk limits, weight range (10 to 100) and active bonus range (0 to 100, default 10) match the `sys ha-group` and `net trunk` tmsh references.

## References

- BIG-IP DSC Administration 14.1, Managing Failover
- Maintaining High Availability Through Resource Monitoring 13.0
- BIG-IP DSC Administration 11.4, Understanding Fast Failover
- tmsh reference, `sys ha-group`
- TMOS Routing Administration, trunk interface limits (K1689, K42303533)

Links to each are in the Help section of the tool.

## License

MIT License. See [LICENSE](LICENSE).

This software is provided as is, without warranty of any kind. The author is not responsible for any outcome of applying configurations derived from it.
