# Operator Configuration

![](Operator-Configuration-1.png)

## Purpose and Scope

This document explains how to configure the LTE redirection attack system for different mobile network operators. The operator configuration is crucial for ensuring the attack can effectively mimic genuine cellular networks, enabling successful redirection of target devices. For information about the underlying network setup, see [Network Configuration](3.1-network-configuration.md), and for details on how the redirector uses these settings, see [LTE Redirector Operation](3.2-lte-redirector-operation.md).

## Operator Selection Process

The system selects which mobile operator to impersonate during the attack through a straightforward configuration mechanism. This selection determines how the `redirect_4_2g` container is configured via telnet commands.

![](Operator-Configuration-2.png)

**Sources:** `scripts/operator` `osmo_egprs/configs/operator` `redirect_4_2g/scripts/operator` `operator`

## Supported Operators

The system is designed to target French mobile networks and supports the following operators:

1. Free Mobile (default)
2. Orange
3. SFR
4. Bouygues Telecom

Each operator has a dedicated telnet configuration script that sets the appropriate parameters for that carrier:

![](Operator-Configuration-3.png)

**Sources:** `scripts/operator` `osmo_egprs/configs/operator` `redirect_4_2g/scripts/operator` `operator`

## Configuration Workflow

The operator configuration is integrated into the overall system setup process:

![](Operator-Configuration-4.png)

**Sources:** `scripts/operator` `osmo_egprs/configs/operator` `redirect_4_2g/scripts/operator` `operator`

## Configuring the Operator

By default, the system is configured for the "Free" operator, as shown in the operator files:

- `operator`
- `osmo_egprs/configs/operator`
- `redirect_4_2g/scripts/operator`
- `scripts/operator`

### Interactive Configuration

When running the attack system via the `run.sh` script, you will be prompted to select the operator you wish to target. This selection is stored in the operator files and used to determine which telnet configuration script to execute.

The telnet configuration occurs after the containers are started, allowing you to dynamically configure the LTE redirector with the appropriate operator settings.

### Manual Configuration

You can also manually set the operator by editing the operator files directly. Simply replace the content of these files with the desired operator name (e.g., "free", "orange", "sfr", or "bouygues").

## Configuration Parameters

Each telnet configuration script sets parameters appropriate for the selected operator, including:

1. Mobile Country Code (MCC)
2. Mobile Network Code (MNC)
3. Network name
4. Cell parameters
5. Frequency bands

These parameters ensure that the fake base station appears as a legitimate network operated by the selected carrier.

| Parameter       | Description                        | Configuration Impact                                |
| --------------- | ---------------------------------- | --------------------------------------------------- |
| MCC             | Mobile Country Code                | Identifies the country (208 for France)             |
| MNC             | Mobile Network Code                | Identifies the specific operator within the country |
| Network Name    | Operator name displayed on devices | Visual identification for users                     |
| Cell Parameters | Cell ID, TAC, etc.                 | Technical identifiers for the cell                  |
| Frequency Bands | Operating frequencies              | Determines which devices can connect                |

**Sources:** `scripts/operator` `osmo_egprs/configs/operator` `redirect_4_2g/scripts/operator` `operator`

## Impact on Attack Process

The operator configuration directly impacts the effectiveness of the attack:

![](Operator-Configuration-5.png)

**Sources:** `scripts/operator` `osmo_egprs/configs/operator` `redirect_4_2g/scripts/operator` `operator`

### Target Selection

Mobile devices prioritize connecting to their home network. By configuring the system to impersonate a specific operator, the attack targets subscribers of that operator.

### Network Visibility

The telnet configuration sets the broadcast parameters that make the fake base station visible to nearby devices. These parameters must match the expected values for the selected operator.

### Authentication Process

Different operators may use different authentication mechanisms. The operator configuration ensures that the system responds appropriately during the initial connection phase.

## Integration with System Components

The operator configuration integrates with other system components to create a complete attack system:

![](Operator-Configuration-6.png)

**Sources:** `scripts/operator` `osmo_egprs/configs/operator` `redirect_4_2g/scripts/operator` `operator`

## Target Operator Selection Strategy

When deciding which operator to target, consider the following factors:

1. **Prevalence**: Target the most common operator in your area for maximum potential targets
2. **Security Measures**: Some operators may implement additional security measures
3. **Network Parameters**: Familiarity with the operator's network parameters can help in fine-tuning the attack

## Testing and Verification

After configuring the operator, you should verify that:

1. The fake base station is broadcasting with the correct operator identifiers
2. Target devices can see the fake network
3. Target devices attempt to connect to the fake network
4. The redirection process works as expected

## Troubleshooting

If targets are not connecting to the fake base station, consider the following:

1. **Verify Operator Selection**: Ensure the correct operator is selected for the target devices
2. **Check Configuration Files**: Confirm that all operator files contain the correct operator name
3. **Telnet Configuration**: Verify that the telnet configuration script executed successfully
4. **Signal Strength**: Ensure the SDR hardware is providing adequate signal strength

## Related Configuration

For information about network configuration related to the operator settings, see [Network Configuration](3.1-network-configuration.md).

For details on how the LTE redirector operates with these settings, see [LTE Redirector Operation](3.2-lte-redirector-operation.md).

*Source: [DeepWiki LTE-Redirection_Attack Operator Configuration](https://deepwiki.com/AidasDir/LTE-Redirection_Attack/5-operator-configuration)* 