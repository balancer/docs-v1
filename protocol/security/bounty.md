# This page has been deprecated. V1 documentation is partially maintained [here](https://docs.balancer.fi/v/v1/protocol/security/bounty)

# Bug Bounty

### Overview

Balancer has completed smart contract audits with Trail of Bits and Open Zeppelin. We also will run a continuous bug bounty program for the bronze release of Balancer core.

### Scope

The bug bounty covers any of the core smart contracts deployed on Mainnet. The code can be found at: [https://github.com/balancer-labs/balancer-core](https://github.com/balancer-labs/balancer-core)

Submissions should be based off commit hash: [https://github.com/balancer-labs/balancer-core/tree/2d88257fb27ad3c84b5166304a342e66055a81b3](https://github.com/balancer-labs/balancer-core/tree/2d88257fb27ad3c84b5166304a342e66055a81b3)

Mainnet BFactory can be found at: [https://etherscan.io/address/0x9424b1412450d0f8fc2255faf6046b98213b76bd](https://etherscan.io/address/0x9424b1412450d0f8fc2255faf6046b98213b76bd)

_Additional second layer contracts, such as the exchange proxy or individual smart pool contracts, may be added at a further date._

### Rewards

The bounty program will pay out rewards according to the severity of a vulnerability. The final reward amount is at the sole discretion of Balancer Labs. See eligibility section below for more details.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Reward</th>
      <th style="text-align:left">Severity</th>
      <th style="text-align:left">Examples</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>$20,000 - $50,000</b>
      </td>
      <td style="text-align:left">Critical</td>
      <td style="text-align:left">
        <ul>
          <li>Stealing assets from a pool</li>
          <li>Permanently freezing pool assets</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>$10,000 - $20,000</b>
      </td>
      <td style="text-align:left">High</td>
      <td style="text-align:left">
        <ul>
          <li>Severe rounding errors where an attacker can steal significant funds in
            excess of any gas costs or swap fees</li>
          <li>Manipulating a finalized pool&apos;s assets / weights / fees</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>$2,000 - $5,000</b>
      </td>
      <td style="text-align:left">Medium</td>
      <td style="text-align:left">
        <ul>
          <li>Minor rounding errors that allow an attacker to slowly manipulate funds
            to their advantage</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>$0 - $2,000</b>
      </td>
      <td style="text-align:left">Low</td>
      <td style="text-align:left">
        <ul>
          <li>Informational and code quality based disclosures</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

### Reporting / Disclosures

Please report any findings to [security@balancer.finance](mailto:security@balancer.finance), with full details about any vulnerability and steps / code to reproduce. Allow us time to review and remediate any findings before public disclosure.

### Ineligible Findings

* Duplicate vulnerabilities. Only the first reporter will be rewarded.
* Findings already known as part of a formal audit.
* Findings related to non-standard ERC20 tokens might be ineligible as many vulnerabilities might be inserted in non-standard ERC20 tokens on purpose for applying for this bug bounty. 

