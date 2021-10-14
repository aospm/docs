 # Qualcomm SMB2 charger documentation
Derived from downstream kernel sources
- [[#About|About]]
- [[#Acronyms|Acronyms]]
- [[#Downstream DT bindings|Downstream DT bindings]]
	- [[#The peripherals|The peripherals]]
		- [[#CHGR|CHGR]]
		- [[#OTG|OTG]]
		- [[#BAT-IF|BAT-IF]]
		- [[#USBIN|USBIN]]
- [[#Register maps|Register maps]]

## About

From downstream dt-binding docs:
> SMB2 Charger is an efficient programmable battery charger capable of charging a
high-capacity lithium-ion battery over micro-USB or USB Type-C ultrafast with
Quick Charge 2.0, Quick Charge 3.0, and USB Power Delivery support. Wireless
charging features full A4WP Rezence 1.2, WPC 1.2, and PMA support.

The smb2 charger is extremely feature rich and offers a lot of complexity in implementation. It is designed to be used in tandem with a fuel gauge and even external charger chips to manage battery charging, power delivery as well as battery health / thermals. It supports USB-C PD chargers and can manage VBUS / VCONN regulators for negotiating host mode and powering external peripherals. It can also manage wireless charging with WIPWR chargers (?).

## Acronyms and Definitions

| Acronym | Meaning                                                                                                                                       |
| ------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| DFP     | Downstream facing port (acting as host)                                                                                                       |
| UFP     | Upstream facing port (acting as peripheral)                                                                                                   |
| CFG     | Configuration                                                                                                                                 |
| SOC     | State of Charge                                                                                                                               |
| SDP     | Standard downstream port (500mA for USB 2.0, 900mA for USB 3.0)                                                                               |
| CDP     | Charding downstream port which supports a high current mode (up to 1.5A)                                                                      |
| DCP     | Dedicated charging port - A wall charger, up to 1.5A                                                                                          |
| OCP     | Over current protection                                                                                                                       |
| OVP     | Over voltage protection                                                                                                                       |
| APSD    | Automatic power source detection (a Qcom designed feature of SMB2)                                                                            |
| ICL     | Current limit                                                                                                                                 |
| QC 2/3  | Qualcomm Quick charge v2 / v3                                                                                                                 |
| OTG     | On The Go (The concept of attaching a USB peripheral like a memory stick to your device when "on the go")                                     |
| Vfloat  | The voltage at which the battery should be maintained after charging is complete. See [Wikiedia](https://en.wikipedia.org/wiki/Float_voltage) |
| SE      | Self Explanatory                                                                                                                                              |

## Downstream DT bindings
The smb2 charger is made up of multiple "peripherals" though not all of them are optional, here is an example DT:
```c
pmi8998_charger: qcom,qpnp-smb2 {
	compatible = "qcom,qpnp-smb2";
	#address-cells = <1>;
	#size-cells = <1>;

	io-channels = <&pmic_rradc 0>;
	io-channel-names = "rradc_batt_id";

	dpdm-supply = <&qusb_phy0>;

	qcom,chgr@1000 {
		reg = <0x1000 0x100>;
		interrupts =    <0x2 0x10 0x0 IRQ_TYPE_NONE>,
				<0x2 0x10 0x1 IRQ_TYPE_NONE>,
				<0x2 0x10 0x2 IRQ_TYPE_NONE>,
				<0x2 0x10 0x3 IRQ_TYPE_NONE>,
				<0x2 0x10 0x4 IRQ_TYPE_NONE>;

		interrupt-names =       "chg-error",
					"chg-state-change",
					"step-chg-state-change",
					"step-chg-soc-update-fail",
					"step-chg-soc-update-request";
	};

	qcom,otg@1100 {
		reg = <0x1100 0x100>;
		interrupts =    <0x2 0x11 0x0 IRQ_TYPE_NONE>,
				<0x2 0x11 0x1 IRQ_TYPE_NONE>,
				<0x2 0x11 0x2 IRQ_TYPE_NONE>,
				<0x2 0x11 0x3 IRQ_TYPE_NONE>;

		interrupt-names =       "otg-fail",
					"otg-overcurrent",
					"otg-oc-dis-sw-sts",
					"testmode-change-detect";
	};

	qcom,bat-if@1200 {
		reg = <0x1200 0x100>;
		interrupts =    <0x2 0x12 0x0 IRQ_TYPE_NONE>,
				<0x2 0x12 0x1 IRQ_TYPE_NONE>,
				<0x2 0x12 0x2 IRQ_TYPE_NONE>,
				<0x2 0x12 0x3 IRQ_TYPE_NONE>,
				<0x2 0x12 0x4 IRQ_TYPE_NONE>,
				<0x2 0x12 0x5 IRQ_TYPE_NONE>;

		interrupt-names =       "bat-temp",
					"bat-ocp",
					"bat-ov",
					"bat-low",
					"bat-therm-or-id-missing",
					"bat-terminal-missing";
	};

	qcom,usb-chgpth@1300 {
		reg = <0x1300 0x100>;
		interrupts =    <0x2 0x13 0x0 IRQ_TYPE_NONE>,
				<0x2 0x13 0x1 IRQ_TYPE_NONE>,
				<0x2 0x13 0x2 IRQ_TYPE_NONE>,
				<0x2 0x13 0x3 IRQ_TYPE_NONE>,
				<0x2 0x13 0x4 IRQ_TYPE_NONE>,
				<0x2 0x13 0x5 IRQ_TYPE_NONE>,
				<0x2 0x13 0x6 IRQ_TYPE_NONE>,
				<0x2 0x13 0x7 IRQ_TYPE_NONE>;

		interrupt-names =       "usbin-collapse",
					"usbin-lt-3p6v",
					"usbin-uv",
					"usbin-ov",
					"usbin-plugin",
					"usbin-src-change",
					"usbin-icl-change",
					"type-c-change";
	};

	qcom,dc-chgpth@1400 {
		reg = <0x1400 0x100>;
		interrupts =    <0x2 0x14 0x0 IRQ_TYPE_NONE>,
				<0x2 0x14 0x1 IRQ_TYPE_NONE>,
				<0x2 0x14 0x2 IRQ_TYPE_NONE>,
				<0x2 0x14 0x3 IRQ_TYPE_NONE>,
				<0x2 0x14 0x4 IRQ_TYPE_NONE>,
				<0x2 0x14 0x5 IRQ_TYPE_NONE>,
				<0x2 0x14 0x6 IRQ_TYPE_NONE>;

		interrupt-names =       "dcin-collapse",
					"dcin-lt-3p6v",
					"dcin-uv",
					"dcin-ov",
					"dcin-plugin",
					"div2-en-dg",
					"dcin-icl-change";
	};

	qcom,chgr-misc@1600 {
		reg = <0x1600 0x100>;
		interrupts =    <0x2 0x16 0x0 IRQ_TYPE_NONE>,
				<0x2 0x16 0x1 IRQ_TYPE_NONE>,
				<0x2 0x16 0x2 IRQ_TYPE_NONE>,
				<0x2 0x16 0x3 IRQ_TYPE_NONE>,
				<0x2 0x16 0x4 IRQ_TYPE_NONE>,
				<0x2 0x16 0x5 IRQ_TYPE_NONE>,
				<0x2 0x16 0x6 IRQ_TYPE_NONE>,
				<0x2 0x16 0x7 IRQ_TYPE_NONE>;

		interrupt-names =       "wdog-snarl",
					"wdog-bark",
					"aicl-fail",
					"aicl-done",
					"high-duty-cycle",
					"input-current-limiting",
					"temperature-change",
					"switcher-power-ok";
	};
};

```

## The peripherals (TODO)

Each peripheral / function loosely wraps a functional part of the chip, they are as follows:

### CHGR

This peripheral defines the general SMB2 base and general interrupts.

### OTG

Responsible for handling USB OTG devices, managing the VBUS regulator and configuring DFP mode.

### BAT-IF

The BATIF peripheral contains some registers to handle battery over current protection and configuring battery charging.

### USBIN

The USBIN peripheral contains a lot of registers for configuring Qualcomm Quick charging, port role management (UFP / DFP).

## Registers
This is not a full register map, many registers are unused or don't have enough context to be properly documented.

### CHGR (general) registers
// TODO: TOC here

#### CHARGING_ENABLE_CMD_REG
##### Address
0x42

##### Description
Enable / disable charging
##### Bit map

| Bits | Name                    | Description               |
| ---- | ----------------------- | ------------------------- |
| 0    | CHARGING_ENABLE_CMD_BIT | Enable / disable charging | 

#### CHGR_CFG2_REG

| Bits | Name                         | Description                                                                         |
| ---- | ---------------------------- | ----------------------------------------------------------------------------------- |
| [7]  | CHG_EN_SRC_BIT               | Enable hw control ??                                                                |
| [6]  | CHG_EN_POLARITY_BIT          | Enable hw control ??                                                                |
| [5]  | PRETOFAST_TRANSITION_CFG_BIT | Unused                                                                              |
| [4]  | BAT_OV_ECC_BIT               | Unused                                                                              |
| [3]  | I_TERM_BIT                   | Unused                                                                              |
| [2]  | AUTO_RECHG_BIT               | Unused                                                                              |
| [1]  | EN_ANALOG_DROP_IN_VBATT_BIT  | Unused                                                                              |
| [0]  | CHARGER_INHIBIT_BIT          | Enable / disable inhibiting charge when withing a specific threshold of [[#Vfloat]] |

#### FAST_CHARGE_CURRENT_CFG_REG
This might be QC fast charge specific ?
##### Address
0x61
##### Bit map
| Bits  | Name    | Description                 |
| ----- | ------- | --------------------------- |
| [7:0] | Current | Maximum fast charge current | 

```c
/* Configure charge enable for software control; active high */
	rc = smblib_masked_write(chg, CHGR_CFG2_REG,
				 CHG_EN_POLARITY_BIT |
				 CHG_EN_SRC_BIT, 0);
```

#### BATTERY_CHARGER_STATUS_1_REG
Some misc battery status flags.

##### Address
0x06
##### Bitmap
| Bits  | Name                  | Description                                 |
| ----- | --------------------- | ------------------------------------------- |
| [7]   | BVR_INITIAL_RAMP_BIT  | Unused                                      |
| [6]   | CC_SOFT_TERMINATE_BIT | Used as part of `op_check_charger_collapse` |
| [5:3] | STEP_CHARGING_STATUS  |                                             |

## Interrupts

The SMB2 charger has a lot of interrupts, many of them are unused and some have really strange handlers. Only the relevant ones are documented below.


| Symbol                | Handler                         | Description                                                                                                    |
| --------------------- | ------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| CHG_ERROR_IRQ         | smblib_handle_debug             | Prints and error message to report that the IRQ was fired.                                                     |
| CHG_STATE_CHANGE_IRQ  | smblib_handle_chg_state_change  | Reads [[#BATTERY_CHARGER_STATUS_1_REG]] to see if the status is TERMINATE_CHARGE and sets chg->chg_done = true |
| OTG_FAIL_IRQ          | smblib_handle_debug             | -                                                                                                              |
| OTG_OVERCURRENT_IRQ   | smblib_handle_otg_overcurrent   | SE -- Reads [[#OTG INT_RT_STS_OFFSET]] and checks for OTG_OVERCURRENT_RT_STS_BIT, resetting OTG if set.        |
| OTG_OC_DIS_SW_STS_IRQ | smblib_handle_debug             | ?                                                                                                              |
| BATT_TEMP_IRQ         | smblib_handle_batt_temp_changed | Calls smblib_recover_from_soft_jeita                                                                                                               |
