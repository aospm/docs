\ # Qualcomm SMB2 charger documentation
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

## Preliminary Reading
+ [The Basics of USB Battery Charging: a Survival Guide](https://theengineer.markallengroup.com/production/content/uploads/2013/03/AN4803.pdf)
+ [Li-ion battery-charger solutions for JEITA compliance](https://www.ti.com/lit/an/slyt365/slyt365.pdf)
+ [What's the role of the CC pin in Type-C solution](https://community.silabs.com/s/article/what-s-the-role-of-cc-pin-in-type-c-solution?language=en_US)

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
| HVDCP   | High Voltage Dedicated Charging Port - a 9v or 12v wall charger                                                                                                                                             |
| OCP     | Over current protection                                                                                                                       |
| OVP     | Over voltage protection                                                                                                                       |
| APSD    | Automatic power source detection (a Qcom designed feature of SMB2)                                                                            |
| ICL     | Current limit                                                                                                                                 |
| QC 2/3  | Qualcomm Quick charge v2 / v3                                                                                                                 |
| OTG     | On The Go (The concept of attaching a USB peripheral like a memory stick to your device when "on the go")                                     |
| Vfloat  | The voltage at which the battery should be maintained after charging is complete. See [Wikiedia](https://en.wikipedia.org/wiki/Float_voltage) |
| SE      | Self Explanatory                                                                                                                              |
| JEITA   | Japan Eleectronics and Information Technology Industries Association - the standard for Li-ion battery safety.                                |

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
Pomodoro
## Registers
This is not a full register map, many registers are unused or don't have enough context to be properly documented.

### CHGR (general) registers
// TODO: TOC here
Pomodoro
#### #CHARGING_ENABLE_CMD_REG
##### Address
0x42

##### Description
Enable / disable charging
##### Bit map

| Bits | Name                    | Description               |
| ---- | ----------------------- | ------------------------- |
| 0    | CHARGING_ENABLE_CMD_BIT | Enable / disable charging | 

#### #CHGR_CFG2_REG

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

#### #BATTERY_CHARGER_STATUS_1_REG
Some misc battery status flags.

##### Address
0x06
##### Bitmap
| Bits  | Name                        | Description                                 |
| ----- | --------------------------- | ------------------------------------------- |
| [7]   | BVR_INITIAL_RAMP_BIT        | Unused                                      |
| [6]   | CC_SOFT_TERMINATE_BIT       | Used as part of `op_check_charger_collapse` |
| [5:3] | STEP_CHARGING_STATUS        | Unused                                      |
| [2:0] | BATTERY_CHARGER_STATUS_MASK | One of: `TRICKLE_CHARGE = 0, PRE_CHARGE, FAST_CHARGE, FULLON_CHARGE, TAPER_CHARGE, TERMINATE_CHARGE, INHIBIT_CHARGE, DISABLE_CHARGE`                                            |


#### #TYPE_C_STATUS_1_REG 
##### Address
0x30B
##### Bitmap


## Interrupts

The SMB2 charger has a lot of interrupts, many of them are unused and some have really strange handlers. Only the relevant ones are documented below.


| Symbol                  | Handler                          | Description                                                                                                                    | Is wakeup? |
| ----------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ---------- |
| #CHG_ERROR_IRQ          | #smblib_handle_debug             | Prints an error message to report that the IRQ was fired.                                                                      |            |
| #CHG_STATE_CHANGE_IRQ   | #smblib_handle_chg_state_change  | Reads [[#BATTERY_CHARGER_STATUS_1_REG]] to see if the status is TERMINATE_CHARGE and sets `chg->chg_done = true`               |            |
| #OTG_FAIL_IRQ           | #smblib_handle_debug             | -                                                                                                                              |            |
| #OTG_OVERCURRENT_IRQ    | #smblib_handle_otg_overcurrent   | Reads [[#OTG INT_RT_STS_OFFSET]] and checks for OTG_OVERCURRENT_RT_STS_BIT, resetting OTG if set.                              |            |
| #OTG_OC_DIS_SW_STS_IRQ  | #smblib_handle_debug             | ?                                                                                                                              |            |
| #BATT_TEMP_IRQ          | #smblib_handle_batt_temp_changed | Calls smblib_recover_from_soft_jeita which handles hitting "soft" JEITA temp limits. and restarts charging.                    |            |
| #BATT_OCP_IRQ           | #smblib_handle_batt_psy_changed  | Calls `power_supply_changed` to propagate the hw change to userspace.                                                          |            |
| #BATT_OV_IRQ            | #Same as above                   | Same as above                                                                                                                  |            |
| #BATT_LOW_IRQ           | #Same as above                   | Same as above                                                                                                                  |            |
| #BATT_THERM_ID_MISS_IRQ | #Same as above                   | Same as above                                                                                                                  |            |
| #BATT_TERM_MISS_IRQ     | #Same as above                   | Same as above                                                                                                                  |            |
| #USBIN_COLLAPSE_IRQ     | #smblib_handle_debug             | Prints an error message to report that the IRQ was fired.                                                                      |            |
| #USBIN_LT_3P6V_IRQ      | #Same as above                   | Same as above (less than 3.6v ?)                                                                                               |            |
| #USBIN_UV_IRQ           | #smblib_handle_usbin_uv          | Only relevant on QC platforms, on OnePlus dash chargers returns immediately. QC seems to drop 12v chargers to 9v and 9v to 5v? |            |
| #USBIN_OV_IRQ           | #smblib_handle_debug             | Prints an error message to report that the IRQ was fired.                                                                      |            |
| #USBIN_PLUGIN_IRQ       | #smblib_handle_usb_plugin        | calls `smblib_usb_plugin_locked` which handles configuring charging. (big and complicated)                                     | x          |
| #USBIN_SRC_CHANGE_IRQ   | #smblib_handle_usb_source_change | Reads APSD and handles the case where the charger has initialised QC / HVDCP.                                                  | x          |
| #USBIN_ICL_CHANGE_IRQ   | #smblib_handle_icl_change        |                                                                                                                                | X          |
| #TYPE_C_CHANGE_IRQ      | #smblib_handle_usb_typec_change  | Causes the #TYPE_C_STATUS_1_REG through 5 registers to be read, these are used to adjust behaviour whilst charging.            | X          |

## Parallel charging

https://github.com/LineageOS/android_kernel_oneplus_sdm845/commit/1d1034c0cef5195c3d957f333e7c33eab60bc2b5
```
Parallel charging increases charging capacity and efficiency by
distributing the current between two charging chips.

PMI8998 feeds the parallel charger via its MID input, and handles
input current limiting in its front-porch FET. As master charger,
PMI8998 is responsible for enabling/disabling the parallel
charger, and the FCC distribution.

To enable parallel charging in software, the following conditions
must be met:
- Strong USBIN input
- Battery present
- In fast or taper charging state
- Attached UFP source
While the enabling/disabling is always under the control of software
the disabling can also be done by hardware in case of fault.

Battery current is usually fixed to the battery rating. The FCC
distribution is simple, a split of 50/50 by default, which can
be changed in runtime.

When taper irq kicks in, the algorithm reduces parallel FCC by 25%.
This puts the charging back in constant current phase until the
next one happens where again the algorithm reduces the FCC by
25%. This continues until the parallel FCC drops to 500mA. At that
time parallel charging is disabled and master continues charging
the rest of constant voltage phase.
```

smb2 sets `chg->mode = PARALLEL_MASTER`, the implementation seems to expect some other driver to probe and register itself as the "slave" charger

## APSD

APSD or Automatic Power Source Detection is a custom hardware implementation by Qualcomm included in the smb2 charger. It automatically figures out what type of cable has been attached to simplify the software side. Usually it is polled multiple times until it has completed (at which point it sets the #APSD_DTC_STATUS_DONE_BIT flag in #APSD_STATUS_REG).

## High level overview

### #USBIN_PLUGIN_IRQ

Attaching the USB cable caused the #USBIN_UV_IRQ to fire, this gets the APSD result by reading #APSD_STATUS_REG (which is 0 as the cable was only just attached).
#USBIN_PLUGIN_IRQ also fires around this time, `smblib_usb_plugin_locked` is called after obtaining a lock, the OnePlus driver has an addiiton to check if fast charging has already started in which case it returns immediately.

The #USBIN_INT_RT_STS register is read to check if the event was a plug-in or unplug and the appropriate frequency is written to #CFG_BUCKBOOST_FREQ_SELECT_BUCK_REG.
> #note Strangely the `freq_removal` frequency is higher than the 5v frequency?

Next, `__pm_stay_awake()` is called if this was a plug-in even to prevent the device from suspending during charge. If this was an unplug event, `__pm_relax()` is called instead.

#### Attach



### Type-C handling

Handling Type-C cables requires a lot of complexity in the devices, in the case of a simple USB-C to A cable this can usually be ignored. But we have to know enough to make sure we don't impede the behaviour of the charger.

The #TYPE_C_CHANGE_IRQ fires whenever there is a change on the type-c port, the type-c status registers are then dumped and stored in `chg->typec_status` so that they can be used in the driver. 

## Downstream

### DCP Charger
Here are dmesg logs from attaching a regular wall charger to the device:
```
[  683.837589] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  683.837667] SMBLIB: smblib_usb_plugin_locked: acquire chg_wake_lock
[  683.837676] SMBLIB: smblib_handle_usbin_uv: PMI: smblib_handle_usbin_uv: DEBUG: RESET STORM COUNT FOR POWER_OK
[  683.839639] SMBLIB: smblib_awake_vote_callback: set awake=1
[  683.839727] SMBLIB: smblib_usb_plugin_locked: IRQ: attached
[  683.964780] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  683.970433] healthd: battery l=100 v=4319 t=29.3 h=2 st=4 c=202 fc=2878000 cc=0 chg=
[  684.080762] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  684.083596] healthd: battery l=100 v=4319 t=29.3 h=2 st=4 c=202 fc=2878000 cc=0 chg=
[  684.489175] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=500000
[  684.489278] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  684.646582] SMBLIB: smblib_handle_usb_source_change: APSD_STATUS=0x01
[  684.646710] SMBLIB: smblib_set_icl_current: icl_ua=475000
[  684.646789] SMBLIB: smblib_set_usb_suspend: suspend=0
[  684.646815] QCOM-BATT: usb_icl_vote_callback: total_icl_ua=1500000
[  684.646826] SMBLIB: smblib_set_icl_current: icl_ua=1500000
[  684.647097] SMBLIB: smblib_set_usb_suspend: suspend=0
[  684.647131] SMBLIB: op_charging_en: enable=1
[  684.647168] SMBLIB: handle_batt_temp_normal: triggered
[  684.647182] SMBLIB: set_chg_ibat_vbat_max: set ibatmax=1950 and set vbatmax=4400
[  684.647198] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  684.647241] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  684.647291] SMBLIB: op_battery_temp_region_set: set temp_region=5
[  684.647314] SMBLIB: smblib_handle_apsd_done: apsd result=0x8, name=DCP, psy_type=5
[  684.647338] SMBLIB: smblib_handle_apsd_done: apsd done,current_now=202
[  684.673292] healthd: battery l=100 v=4327 t=29.3 h=2 st=2 c=128 fc=2878000 cc=0 chg=a
[  684.673326] healthd: yfb '1'
[  684.675851] synaptics,s3320: changer_write_func:ts->changer_connet = 1
[  684.855294] healthd: battery l=100 v=4327 t=29.3 h=2 st=2 c=128 fc=2878000 cc=0 chg=a
[  684.886706] healthd: battery l=100 v=4327 t=29.3 h=2 st=2 c=128 fc=2878000 cc=0 chg=a
[  685.141450] SMBLIB: set_usb_switch: switch on fastchg
[  685.151577] FASTCHG: usb_sw_gpio_set: set usb_sw_gpio=1
[  685.151621] FASTCHG: usb_sw_gpio_set: get usb_sw_gpio=1&1
[  685.367697] healthd: battery l=100 v=4339 t=29.3 h=2 st=2 c=-121 fc=2878000 cc=0 chg=a
[  685.862130] healthd: battery l=100 v=4339 t=29.3 h=2 st=2 c=-121 fc=2878000 cc=0 chg=a
[  686.951417] healthd: battery l=100 v=4377 t=29.4 h=2 st=2 c=-329 fc=2878000 cc=0 chg=a
[  687.686264] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  687.686412] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  690.005429] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  690.005505] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  690.019468] healthd: battery l=100 v=4379 t=29.4 h=2 st=2 c=-321 fc=2878000 cc=0 chg=a
[  693.371527] SMBLIB: retrigger_dash_work: retrger dash
[  693.371576] SMBLIB: set_usb_switch: switch off fastchg
[  693.371596] FASTCHG: usb_sw_gpio_set: set usb_sw_gpio=0
[  693.371652] FASTCHG: usb_sw_gpio_set: get usb_sw_gpio=0&0
[  693.371674] SMBLIB: set_usb_switch: switch on fastchg
[  693.381808] FASTCHG: usb_sw_gpio_set: set usb_sw_gpio=1
[  693.381854] FASTCHG: usb_sw_gpio_set: get usb_sw_gpio=1&1
[  693.614781] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  693.614855] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  693.991070] healthd: battery l=100 v=4379 t=29.5 h=2 st=2 c=-312 fc=2878000 cc=0 chg=a

<these messages are printed regularly>

[  700.662474] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  700.662548] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  701.049995] healthd: battery l=100 v=4380 t=29.6 h=2 st=2 c=-297 fc=2878000 cc=0 chg=a
[  707.107761] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  707.107862] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  707.467937] healthd: battery l=100 v=4381 t=29.6 h=2 st=2 c=-290 fc=2878000 cc=0 chg=a
[  713.457816] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  713.457916] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  713.824453] healthd: battery l=100 v=4381 t=29.7 h=2 st=2 c=-284 fc=2878000 cc=0 chg=a
[  714.080171] SMBLIB: smblib_awake_vote_callback: set awake=0
[  716.241095] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  716.241172] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  716.250741] healthd: battery l=100 v=4381 t=29.7 h=2 st=2 c=-281 fc=2878000 cc=0 chg=a
[  720.488569] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  720.488647] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
```
Disconnecting:
```
[ 1557.472267] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1557.472320] SMBLIB: smblib_handle_usbin_uv: PMI: smblib_handle_usbin_uv: DEBUG: RESET STORM COUNT FOR POWER_OK
[ 1557.472532] SMBLIB: smblib_usb_plugin_locked: release chg_wake_lock
[ 1557.473055] SMBLIB: set_usb_switch: switch off fastchg
[ 1557.473070] FASTCHG: usb_sw_gpio_set: set usb_sw_gpio=0
[ 1557.473116] FASTCHG: usb_sw_gpio_set: get usb_sw_gpio=0&0
[ 1557.473172] SMBLIB: op_get_aicl_work: AICL result=0mA
[ 1557.473204] SMBLIB: set_dash_charger_present: dash_present = 0, charger_present = 0
[ 1557.473239] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=500000
[ 1557.473287] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[ 1557.473348] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1557.473378] SMBLIB: op_battery_temp_region_set: set temp_region=8
[ 1557.478989] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=500000
[ 1557.479032] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[ 1557.530521] SMBLIB: smblib_usb_plugin_locked: IRQ: detached
[ 1557.530945] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1557.531036] SMBLIB: smblib_set_icl_current: icl_ua=25000
[ 1557.531050] SMBLIB: smblib_set_usb_suspend: suspend=1
[ 1557.531109] QCOM-BATT: usb_icl_vote_callback: total_icl_ua=500000
[ 1557.531127] SMBLIB: smblib_set_icl_current: icl_ua=500000
[ 1557.531187] SMBLIB: smblib_set_usb_suspend: suspend=0
[ 1557.531714] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1557.590698] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1557.606729] healthd: battery l=100 v=4352 t=28.8 h=2 st=3 c=0 fc=2878000 cc=0 chg=
[ 1557.606757] healthd: yfb '0'
[ 1557.608485] synaptics,s3320: changer_write_func:ts->changer_connet = 0
[ 1557.650430] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1557.658650] healthd: battery l=100 v=4352 t=28.8 h=2 st=3 c=0 fc=2878000 cc=0 chg=
[ 1557.735147] healthd: battery l=100 v=4352 t=28.8 h=2 st=3 c=0 fc=2878000 cc=0 chg=
```
#### With full register read/write logging
```
[  161.508186] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 0)
[  161.508226] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  161.508248] SMBLIB: smblib_handle_usbin_uv: PMI: smblib_handle_usbin_uv: DEBUG: RESET STORM COUNT FOR POWER_OK
[  161.508334] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.508398] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x16a0, val=0x 0)
[  161.508421] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  161.508434] SMBLIB: smblib_usb_plugin_locked: acquire chg_wake_lock
[  161.509962] SMBLIB: smblib_awake_vote_callback: set awake=1
[  161.510224] SMBLIB: smblib_usb_plugin_locked: IRQ: attached
[  161.510342] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.510425] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.510726] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  161.510780] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.510826] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.510874] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1410, val=0x 6)
[  161.510958] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  161.510992] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x47)
[  161.511023] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x47)
[  161.511057] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x47)
[  161.512651] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.512689] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.513002] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  161.514171] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  161.514207] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.514253] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  161.514907] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  161.584673] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_masked_write(addr = 0x1358, mask=0x80, val=0x 0)
[  161.584809] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x16a1, val=0x 1)
[  161.631439] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  161.631510] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1368, val=0x30)
[  161.631572] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  161.631606] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.631722] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 0)
[  161.631735] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  161.631765] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x130f, val=0x 0)
[  161.631794] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  161.633027] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.633401] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  161.634400] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  161.634428] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x47)
[  161.634531] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  161.634812] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  161.635086] healthd: battery l=100 v=4314 t=28.8 h=2 st=4 c=193 fc=2878000 cc=0 chg=
[  161.639339] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  161.639427] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.639462] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.639489] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1410, val=0x 6)
[  161.639542] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  161.639566] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x47)
[  161.639589] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x47)
[  161.639613] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x47)
[  161.639908] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.639940] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.639991] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  161.640557] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  161.640583] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.640632] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  161.640679] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  161.750679] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  161.750768] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1368, val=0x30)
[  161.750844] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  161.750882] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  161.751012] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 0)
[  161.751027] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  161.751060] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x130f, val=0x 0)
[  161.751090] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  161.752438] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  161.754337] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  161.754373] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x47)
[  161.754510] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  161.754886] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  161.755075] healthd: battery l=100 v=4314 t=28.8 h=2 st=4 c=193 fc=2878000 cc=0 chg=
[  162.142414] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x43)
[  162.143271] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.143306] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.143337] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.143368] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.143514] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.143546] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.143576] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.143607] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.143648] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x80)
[  162.143687] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  162.143723] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.144046] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.144081] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.144126] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  162.144160] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x 0)
[  162.144180] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=500000
[  162.144207] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x1061, val=0x14)
[  162.144245] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  162.144260] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x1070, val=0x79)
[  162.144332] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  162.144365] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.144405] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.144440] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1410, val=0x 6)
[  162.144503] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.144534] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.144563] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.144592] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.144623] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.144653] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.144978] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.145299] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x80)
[  162.145608] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x80)
[  162.145641] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.145683] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.145751] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.261433] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.325421] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 1)
[  162.325463] SMBLIB: smblib_handle_usb_source_change: APSD_STATUS=0x01
[  162.325497] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 1)
[  162.325527] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1308, val=0x 8)
[  162.325596] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  162.325614] SMBLIB: smblib_set_icl_current: icl_ua=475000
[  162.325634] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x1370, val=0x13)
[  162.325687] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_masked_write(addr = 0x1366, mask=0x 1, val=0x 1)
[  162.325721] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_masked_write(addr = 0x1365, mask=0x10, val=0x10)
[  162.325748] SMBLIB: smblib_set_usb_suspend: suspend=0
[  162.325764] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_masked_write(addr = 0x1340, mask=0x 1, val=0x 0)
[  162.325792] QCOM-BATT: usb_icl_vote_callback: total_icl_ua=1500000
[  162.325802] SMBLIB: smblib_set_icl_current: icl_ua=1500000
[  162.325818] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x1370, val=0x3c)
[  162.325857] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_masked_write(addr = 0x1366, mask=0x 1, val=0x 1)
[  162.325887] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_masked_write(addr = 0x1365, mask=0x10, val=0x10)
[  162.325911] SMBLIB: smblib_set_usb_suspend: suspend=0
[  162.325926] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_masked_write(addr = 0x1340, mask=0x 1, val=0x 0)
[  162.325959] SMBLIB: op_charging_en: enable=1
[  162.325975] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_masked_write(addr = 0x1042, mask=0x 1, val=0x 1)
[  162.326014] SMBLIB: handle_batt_temp_normal: triggered
[  162.326029] SMBLIB: set_chg_ibat_vbat_max: set ibatmax=1950 and set vbatmax=4400
[  162.326054] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  162.326077] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x1061, val=0x4e)
[  162.326115] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  162.326130] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x1070, val=0x79)
[  162.326182] SMBLIB: op_battery_temp_region_set: set temp_region=5
[  162.326199] SMBLIB: smblib_handle_apsd_done: apsd result=0x8, name=DCP, psy_type=5
[  162.326220] SMBLIB: smblib_handle_apsd_done: apsd done,current_now=193
[  162.326260] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 1)
[  162.326289] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1308, val=0x 8)
[  162.326373] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 1)
[  162.326539] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1070, val=0x79)
[  162.326582] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1061, val=0x4e)
[  162.326623] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x13)
[  162.326666] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x80)
[  162.326702] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1365, val=0xf7)
[  162.326731] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1370, val=0x3c)
[  162.327650] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.328157] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  162.328433] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  162.328470] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.328509] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.328543] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1410, val=0x 6)
[  162.330707] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.330748] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.330778] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.330807] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.336584] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.336619] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  162.339638] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.339672] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.339701] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.339731] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.340991] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.341026] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.341055] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.341083] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.341260] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x80)
[  162.341720] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.342336] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x14)
[  162.342598] healthd: battery l=100 v=4305 t=28.7 h=2 st=2 c=85 fc=2878000 cc=0 chg=a
[  162.342627] healthd: yfb '1'
[  162.344432] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.344466] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.344496] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.344588] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.344677] synaptics,s3320: changer_write_func:ts->changer_connet = 1
[  162.347156] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.347190] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x15)
[  162.352080] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.352114] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.352143] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.352171] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.352203] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.352233] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.352690] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.352784] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x80)
[  162.355421] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x80)
[  162.355453] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.355496] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.355545] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.357943] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.357973] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x16)
[  162.368732] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.368762] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x17)
[  162.379637] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.379675] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x18)
[  162.382386] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  162.382469] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.382524] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.390483] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.390517] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x19)
[  162.401131] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.401163] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1a)
[  162.411973] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.412009] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1b)
[  162.422698] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.422725] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1c)
[  162.433481] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.433503] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1d)
[  162.444278] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.444300] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1e)
[  162.455216] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  162.455238] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1f)
[  162.510497] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1f)
[  162.510572] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1368, val=0x30)
[  162.510638] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1f)
[  162.510676] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.510806] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 1)
[  162.510836] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1308, val=0x 8)
[  162.510870] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x130f, val=0x 0)
[  162.510899] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1f)
[  162.511852] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x 0)
[  162.512673] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  162.522833] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.522870] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.522900] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.522929] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.524937] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.525055] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1606, val=0x 2)
[  162.532075] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.532110] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.532139] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.532169] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.533221] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.533252] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.533280] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.533309] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.533475] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  162.533895] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.534313] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1f)
[  162.534558] healthd: battery l=100 v=4305 t=28.7 h=2 st=2 c=85 fc=2878000 cc=0 chg=a
[  162.535700] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  162.537048] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.537074] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.537096] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.537117] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.548313] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.548347] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.548368] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.548389] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.549412] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.549435] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  162.549455] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  162.549475] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  162.549579] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  162.549801] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  162.550297] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x1f)
[  162.550702] healthd: battery l=100 v=4305 t=28.7 h=2 st=2 c=85 fc=2878000 cc=0 chg=a
[  162.820414] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.820466] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 1)
[  162.820550] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1308, val=0x 8)
[  162.820603] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  162.823215] SMBLIB: set_usb_switch: switch on fastchg
[  162.833320] FASTCHG: usb_sw_gpio_set: set usb_sw_gpio=1
[  162.833366] FASTCHG: usb_sw_gpio_set: get usb_sw_gpio=1&1
[  163.045520] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.045574] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  163.045606] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.045635] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.047720] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.047756] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  163.047788] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.047818] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.048877] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.048913] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  163.048943] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.048974] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.049005] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.050384] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.050419] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  163.050448] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.050495] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.051739] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.051782] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  163.051812] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.051841] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.052065] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  163.052104] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.052148] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.052182] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1410, val=0x 6)
[  163.052989] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.053022] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  163.053051] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.053080] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.054145] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.054176] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  163.054204] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.054232] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.054270] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  163.054306] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  163.054338] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  163.055372] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.055403] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 3)
[  163.055433] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.055462] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.056642] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.057682] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.057716] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.057747] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.057775] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.059831] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.059863] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.059893] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.059923] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.061012] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.061045] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.061074] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.061102] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.061135] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.061165] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.061563] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.061617] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  163.061952] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  163.061984] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.062026] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.062082] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.144366] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.144433] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1307, val=0x 1)
[  163.144466] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1308, val=0x 8)
[  163.161838] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.161875] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.161908] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1340, val=0x 0)
[  163.161941] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1042, val=0x 1)
[  163.209168] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  163.209224] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x20)
[  163.216246] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x40)
[  163.216280] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x21)
[  163.280787] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.402960] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  163.403076] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x 0)
[  163.403130] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.403275] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x 0)
[  163.406844] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.406908] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1606, val=0x 2)
[  163.408892] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  163.410187] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.410217] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.410241] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.410266] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.416354] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.416382] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.416407] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.416431] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.417451] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.417477] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.417501] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.417524] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.417653] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  163.417944] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.418295] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x21)
[  163.418532] healthd: battery l=100 v=4359 t=28.7 h=2 st=2 c=-307 fc=2878000 cc=0 chg=a
[  163.418955] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.418984] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.419008] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.419032] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.421213] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.421242] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.421267] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.421292] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.421421] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  163.421451] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.421484] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.421510] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1410, val=0x 6)
[  163.422399] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.422426] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.422452] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.422476] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.424168] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.424197] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.424222] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.424247] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.426393] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.426423] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.426448] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.426472] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.427711] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.427737] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.427762] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.427786] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.427811] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.427837] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.428215] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.428268] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  163.428607] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  163.428635] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.428673] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.428722] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.429383] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.429412] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.429438] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.429463] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.429499] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  163.429528] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  163.429557] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.430592] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.430621] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.430647] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.430674] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.490385] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.621978] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  163.622083] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x 0)
[  163.622132] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  163.622272] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x 0)
[  163.627008] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.627074] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1606, val=0x 2)
[  163.629659] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  163.631278] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.631315] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.631345] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.631375] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.638689] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.638724] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.638754] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.638783] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.639863] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.639895] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  163.639925] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  163.639954] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  163.640607] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  163.641028] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  163.641578] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x21)
[  163.641894] healthd: battery l=100 v=4359 t=28.7 h=2 st=2 c=-307 fc=2878000 cc=0 chg=a
[  164.246016] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x21)
[  164.246548] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1070, val=0x79)
[  164.246600] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1061, val=0x4e)
[  164.246650] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x21)
[  164.246698] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  164.246738] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1365, val=0xf7)
[  164.246769] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1370, val=0x3c)
[  164.248961] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1210, val=0x 0)
[  164.253877] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  164.253912] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  164.253943] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  164.253972] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  164.260829] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  164.260870] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  164.260900] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  164.260985] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  164.262127] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  164.262162] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  164.262191] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x100d, val=0x70)
[  164.262222] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1044, val=0x 0)
[  164.262399] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
[  164.262791] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160b, val=0x95)
[  164.263195] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x21)
[  164.263438] healthd: battery l=100 v=4378 t=28.7 h=2 st=2 c=-328 fc=2878000 cc=0 chg=a
[  165.366189] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1310, val=0x10)
[  165.366296] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1006, val=0x 4)
[  165.366388] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1607, val=0x21)
[  165.366432] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x160a, val=0x 0)
[  165.366476] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  165.366517] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x1061, val=0x4e)
[  165.366601] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  165.366619] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_write(addr = 0x1070, val=0x79)
[  165.366869] qcom,qpnp-smb2 c440000.qcom,spmi:qcom,pmi8998@2:qcom,qpnp-smb2: smblib_read(addr = 0x1007, val=0x 0)
```
### Attaching a PC USB cable:
```
[ 1588.450423] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1588.450492] SMBLIB: smblib_usb_plugin_locked: acquire chg_wake_lock
[ 1588.450500] SMBLIB: smblib_handle_usbin_uv: PMI: smblib_handle_usbin_uv: DEBUG: RESET STORM COUNT FOR POWER_OK
[ 1588.453842] SMBLIB: smblib_awake_vote_callback: set awake=1
[ 1588.453927] SMBLIB: smblib_usb_plugin_locked: IRQ: attached
[ 1588.574837] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1588.578433] healthd: battery l=100 v=4333 t=28.7 h=2 st=4 c=145 fc=2878000 cc=0 chg=
[ 1588.690651] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1588.694430] healthd: battery l=100 v=4333 t=28.7 h=2 st=4 c=145 fc=2878000 cc=0 chg=
[ 1589.263336] SMBLIB: smblib_handle_usb_source_change: APSD_STATUS=0x01
[ 1589.263513] SMBLIB: smblib_set_icl_current: icl_ua=475000
[ 1589.263558] SMBLIB: set_sdp_current: PMI: set_sdp_current: ICL 475000uA isn't supported for SDP
[ 1589.263600] SMBLIB: set_sdp_current: PMI: set_sdp_current: ICL 475000uA isn't supported for SDP
[ 1589.263614] SMBLIB: smblib_set_icl_current: PMI: smblib_set_icl_current: Couldn't set SDP ICL rc=-22
[ 1589.263627] QCOM-BATT: usb_icl_vote_callback: total_icl_ua=525000
[ 1589.263639] SMBLIB: smblib_set_icl_current: icl_ua=525000
[ 1589.263678] SMBLIB: set_sdp_current: PMI: set_sdp_current: ICL 525000uA isn't supported for SDP
[ 1589.263717] SMBLIB: set_sdp_current: PMI: set_sdp_current: ICL 525000uA isn't supported for SDP
[ 1589.263731] SMBLIB: smblib_set_icl_current: PMI: smblib_set_icl_current: Couldn't set SDP ICL rc=-22
[ 1589.263779] QCOM-BATT: usb_icl_vote_callback: total_icl_ua=500000
[ 1589.263792] SMBLIB: smblib_set_icl_current: icl_ua=500000
[ 1589.263989] SMBLIB: smblib_set_usb_suspend: suspend=0
[ 1589.264020] SMBLIB: op_charging_en: enable=1
[ 1589.264310] SMBLIB: handle_batt_temp_normal: triggered
[ 1589.264328] SMBLIB: set_chg_ibat_vbat_max: set ibatmax=1950 and set vbatmax=4400
[ 1589.264350] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[ 1589.264404] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[ 1589.264471] SMBLIB: op_battery_temp_region_set: set temp_region=5
[ 1589.264496] SMBLIB: smblib_handle_apsd_done: apsd result=0x1, name=SDP, psy_type=4
[ 1589.264541] SMBLIB: smblib_handle_apsd_done: apsd done,current_now=145
[ 1589.267514] usbpd usbpd0: Type-C Source (default) connected
[ 1589.269156] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[ 1589.269201] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[ 1589.278625] msm-dwc3 a600000.ssusb: DWC3 exited from low power mode
[ 1589.288719] healthd: battery l=100 v=4328 t=28.8 h=2 st=4 c=131 fc=2878000 cc=0 chg=u
[ 1589.288743] healthd: yfb '1'
[ 1589.291314] synaptics,s3320: changer_write_func:ts->changer_connet = 1
[ 1589.390377] healthd: battery l=100 v=4328 t=28.8 h=2 st=4 c=131 fc=2878000 cc=0 chg=u
[ 1589.421738] IRQ6 no longer affine to CPU4
[ 1589.566254] android_work: sent uevent USB_STATE=CONNECTED
[ 1589.572007] android_work: sent uevent USB_STATE=DISCONNECTED
[ 1589.583015] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[ 1589.583073] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[ 1589.743835] SMBLIB: notify_usb_enumeration_function: status=1
[ 1589.757895] android_work: sent uevent USB_STATE=CONNECTED
[ 1589.805150] configfs-gadget gadget: high-speed config #1: b
[ 1589.806066] msm-dwc3 a600000.ssusb: Avail curr from USB = 500
[ 1589.806085] SMBLIB: smblib_set_prop_sdp_current_max: set usb current_max=500000
[ 1589.806561] android_work: sent uevent USB_STATE=CONFIGURED
[ 1589.835229] healthd: battery l=100 v=4344 t=28.7 h=2 st=2 c=-5 fc=2878000 cc=0 chg=u
[ 1590.320669] SMBLIB: smbchg_re_det_work: re_det, usb_enum_status
[ 1590.337152] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[ 1590.337217] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[ 1590.537658] healthd: battery l=100 v=4344 t=28.7 h=2 st=2 c=-5 fc=2878000 cc=0 chg=u
```
Disconnecting
```
[ 1648.033134] SMBLIB: smblib_usb_plugin_locked: release chg_wake_lock
[ 1648.033209] SMBLIB: set_usb_switch: switch off fastchg
[ 1648.033219] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1648.033230] SMBLIB: smblib_handle_usbin_uv: PMI: smblib_handle_usbin_uv: DEBUG: RESET STORM COUNT FOR POWER_OK
[ 1648.033252] FASTCHG: usb_sw_gpio_set: set usb_sw_gpio=0
[ 1648.033302] FASTCHG: usb_sw_gpio_set: get usb_sw_gpio=0&0
[ 1648.033386] SMBLIB: set_dash_charger_present: dash_present = 0, charger_present = 0
[ 1648.033406] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=500000
[ 1648.033461] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[ 1648.033524] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1648.033557] SMBLIB: op_battery_temp_region_set: set temp_region=8
[ 1648.034599] msm-dwc3 a600000.ssusb: Avail curr from USB = 2
[ 1648.038747] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=500000
[ 1648.038789] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[ 1648.090370] SMBLIB: smblib_usb_plugin_locked: IRQ: detached
[ 1648.090770] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1648.091151] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1648.151066] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1648.152594] usbpd usbpd0: USB Type-C disconnect
[ 1648.163763] msm-dwc3 a600000.ssusb: DWC3 in low power mode
[ 1648.165164] android_work: sent uevent USB_STATE=DISCONNECTED
[ 1648.171879] healthd: battery l=100 v=4356 t=28.7 h=2 st=3 c=3 fc=2878000 cc=0 chg=
[ 1648.171908] healthd: yfb '0'
[ 1648.173726] synaptics,s3320: changer_write_func:ts->changer_connet = 0
[ 1648.210383] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[ 1648.221280] healthd: battery l=100 v=4356 t=28.7 h=2 st=3 c=3 fc=2878000 cc=0 chg=
[ 1648.282230] healthd: battery l=100 v=4356 t=28.7 h=2 st=3 c=3 fc=2878000 cc=0 chg=
[ 1648.311603] read descriptors
[ 1648.311758] read strings
[ 1648.316036] init: processing action (sys.usb.config=adb && sys.usb.configfs=1 && sys.usb.ffs.ready=1) from (/system/etc/init/hw/init.usb.configfs.rc:20)
[ 1648.319040] init: Command 'symlink /config/usb_gadget/g1/functions/ffs.adb /config/usb_gadget/g1/configs/b.1/f1' action=sys.usb.config=adb && sys.usb.configfs=1 && sys.usb.ffs.ready=1 (/system/etc/init/hw/init.usb.configfs.rc:22) took 0ms and failed: symlink() failed: File exists
[ 1648.321867] android_work: did not send uevent (0 0 0000000000000000)
```

### OnePlus Dash charger
The following is from attaching a OnePlus fast charger (the phone was fully charged)
```
[  871.055948] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  871.056029] SMBLIB: smblib_usb_plugin_locked: acquire chg_wake_lock
[  871.056037] SMBLIB: smblib_handle_usbin_uv: PMI: smblib_handle_usbin_uv: DEBUG: RESET STORM COUNT FOR POWER_OK
[  871.060694] SMBLIB: smblib_awake_vote_callback: set awake=1
[  871.060788] SMBLIB: smblib_usb_plugin_locked: IRQ: attached
[  871.184809] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  871.190536] healthd: battery l=100 v=4329 t=29.2 h=2 st=4 c=141 fc=2878000 cc=0 chg=
[  871.250609] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  871.253127] healthd: battery l=100 v=4329 t=29.2 h=2 st=4 c=141 fc=2878000 cc=0 chg=
[  871.521628] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=500000
[  871.521703] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  871.624356] SMBLIB: smblib_get_apsd_result: APSD_DTC_STATUS_DONE_BIT is 0
[  871.640603] SMBLIB: handle_batt_temp_normal: triggered
[  871.640626] SMBLIB: set_chg_ibat_vbat_max: set ibatmax=1950 and set vbatmax=4400
[  871.640653] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  871.640710] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  871.640763] SMBLIB: op_battery_temp_region_set: set temp_region=5
[  871.738029] SMBLIB: smblib_handle_usb_source_change: APSD_STATUS=0x01
[  871.738155] SMBLIB: smblib_set_icl_current: icl_ua=475000
[  871.738240] SMBLIB: smblib_set_usb_suspend: suspend=0
[  871.738266] QCOM-BATT: usb_icl_vote_callback: total_icl_ua=1500000
[  871.738278] SMBLIB: smblib_set_icl_current: icl_ua=1500000
[  871.738547] SMBLIB: smblib_set_usb_suspend: suspend=0
[  871.738594] SMBLIB: op_charging_en: enable=1
[  871.738651] SMBLIB: handle_batt_temp_normal: triggered
[  871.738665] SMBLIB: set_chg_ibat_vbat_max: set ibatmax=1950 and set vbatmax=4400
[  871.738683] SMBLIB: op_battery_temp_region_set: set temp_region=5
[  871.738697] SMBLIB: smblib_handle_apsd_done: apsd result=0x2, name=OCP, psy_type=5
[  871.738711] SMBLIB: smblib_handle_apsd_done: apsd done,current_now=141
[  871.739976] usbpd usbpd0: Type-C Source (default) connected
[  871.756596] healthd: battery l=100 v=4328 t=29.4 h=2 st=4 c=59 fc=2878000 cc=0 chg=a
[  871.756622] healthd: yfb '1'
[  871.758513] synaptics,s3320: changer_write_func:ts->changer_connet = 1
[  871.953776] healthd: battery l=100 v=4328 t=29.4 h=2 st=4 c=59 fc=2878000 cc=0 chg=a
[  871.967894] healthd: battery l=100 v=4328 t=29.4 h=2 st=4 c=59 fc=2878000 cc=0 chg=a
[  872.238320] SMBLIB: set_usb_switch: switch on fastchg
[  872.248472] FASTCHG: usb_sw_gpio_set: set usb_sw_gpio=1
[  872.248518] FASTCHG: usb_sw_gpio_set: get usb_sw_gpio=1&1
[  872.306529] healthd: battery l=100 v=4328 t=29.4 h=2 st=2 c=59 fc=2878000 cc=0 chg=a
[  873.013811] healthd: battery l=100 v=4343 t=29.5 h=2 st=2 c=-137 fc=2878000 cc=0 chg=a
[  873.060812] FASTCHG: fastcg_work_func:
[  873.242306] FASTCHG: dash_read: recv data:0x52
[  873.249789] dashd: soc=100, temp=29, vbat=4343, ibat=-137
[  873.249913] SMBLIB: set_dash_charger_present: set dash online
[  873.252909] SMBLIB: set_dash_charger_present: dash_present = 1, charger_present = 1
[  873.253017] FASTCHG: dash_dev_ioctl: REJECT_DATA
[  873.330451] FASTCHG: request_mcu_irq: request_mcu_irq
[  873.331410] dashd: dash read
[  873.371533] healthd: battery l=100 v=4343 t=29.5 h=2 st=2 c=-137 fc=2878000 cc=0 chg=a
[  873.386820] FASTCHG: fastcg_work_func:
[  873.508556] healthd: battery l=100 v=4343 t=29.5 h=2 st=2 c=-137 fc=2878000 cc=0 chg=a
[  873.571125] FASTCHG: dash_read: recv data:0x5e
[  873.647549] FASTCHG: request_mcu_irq: request_mcu_irq
[  873.648139] dashd: dash read
[  873.724938] FASTCHG: fastcg_work_func:
[  873.910772] FASTCHG: dash_read: recv data:0x0
[  873.910941] dashd: get_adapter_firmware =0x0
[  873.920343] dashd: soc=100, temp=29, vbat=4378, ibat=-314
[  873.920443] SMBLIB: set_dash_charger_present: dash_present = 1, charger_present = 1
[  873.920597] FASTCHG: dash_dev_ioctl: REJECT_DATA
[  873.994048] FASTCHG: request_mcu_irq: request_mcu_irq
[  873.994723] dashd: dash read
[  874.034636] FASTCHG: fastcg_work_func:
[  874.217927] FASTCHG: dash_read: recv data:0x7f
[  874.228129] FASTCHG: request_mcu_irq: request_mcu_irq
[  874.228658] dashd: dash read
[  874.453472] FASTCHG: fastcg_work_func:
[  874.636755] FASTCHG: dash_read: recv data:0x52
[  874.646220] dashd: soc=100, temp=29, vbat=4380, ibat=-306
[  874.646329] SMBLIB: set_dash_charger_present: dash_present = 1, charger_present = 1
[  874.646450] FASTCHG: dash_dev_ioctl: REJECT_DATA
[  874.720392] FASTCHG: request_mcu_irq: request_mcu_irq
[  874.721089] dashd: dash read
[  874.780572] FASTCHG: fastcg_work_func:
[  874.800495] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  874.800568] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  874.964900] FASTCHG: dash_read: recv data:0x5e
[  875.037661] FASTCHG: request_mcu_irq: request_mcu_irq
[  875.038193] dashd: dash read
[  875.115074] FASTCHG: fastcg_work_func:
[  875.312114] FASTCHG: dash_read: recv data:0x0
[  875.322917] SMBLIB: set_dash_charger_present: dash_present = 1, charger_present = 1
[  875.322934] FASTCHG: dash_dev_ioctl: REJECT_DATA
[  875.325452] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  875.325519] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  875.397630] FASTCHG: request_mcu_irq: request_mcu_irq
[  875.438096] FASTCHG: fastcg_work_func:
[  875.522200] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  875.522280] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  875.616842] FASTCHG: dash_read: recv data:0x7f
[  875.627024] FASTCHG: request_mcu_irq: request_mcu_irq
[  875.839280] FASTCHG: fastcg_work_func:
[  876.032578] FASTCHG: dash_read: recv data:0x52
[  876.042251] SMBLIB: set_dash_charger_present: dash_present = 1, charger_present = 1
[  876.042383] FASTCHG: dash_dev_ioctl: REJECT_DATA
[  876.044788] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  876.044850] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  876.116161] FASTCHG: request_mcu_irq: request_mcu_irq
[  876.173281] FASTCHG: fastcg_work_func:
[  876.308308] healthd: battery l=100 v=4380 t=29.5 h=2 st=2 c=-305 fc=2878000 cc=0 chg=a
[  876.359199] FASTCHG: dash_read: recv data:0x5e
[  876.432010] FASTCHG: request_mcu_irq: request_mcu_irq
[  876.510407] FASTCHG: fastcg_work_func:
[  876.715329] FASTCHG: dash_read: recv data:0x0
[  876.727003] SMBLIB: set_dash_charger_present: dash_present = 1, charger_present = 1
[  876.727136] FASTCHG: dash_dev_ioctl: REJECT_DATA
[  876.729527] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  876.729592] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  876.804499] FASTCHG: request_mcu_irq: request_mcu_irq
[  876.845176] FASTCHG: fastcg_work_func:
[  876.997407] healthd: battery l=100 v=4380 t=29.5 h=2 st=2 c=-300 fc=2878000 cc=0 chg=a
[  877.034811] FASTCHG: dash_read: recv data:0x7f
[  877.044991] FASTCHG: request_mcu_irq: request_mcu_irq
[  877.232078] FASTCHG: fastcg_work_func:
[  877.436359] FASTCHG: dash_read: recv data:0x52
[  877.445095] SMBLIB: set_dash_charger_present: dash_present = 1, charger_present = 1
[  877.445113] FASTCHG: dash_dev_ioctl: REJECT_DATA
[  877.447846] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  877.447910] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  877.518606] FASTCHG: request_mcu_irq: request_mcu_irq
[  877.579307] FASTCHG: fastcg_work_func:
[  877.708725] healthd: battery l=100 v=4380 t=29.5 h=2 st=2 c=-299 fc=2878000 cc=0 chg=a
[  877.760812] FASTCHG: dash_read: recv data:0x5e
[  877.834268] FASTCHG: request_mcu_irq: request_mcu_irq
[  877.913277] FASTCHG: fastcg_work_func:
[  878.104395] FASTCHG: dash_read: recv data:0x0
[  878.113522] SMBLIB: set_dash_charger_present: dash_present = 1, charger_present = 1
[  878.113659] FASTCHG: dash_dev_ioctl: REJECT_DATA
[  878.116250] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  878.116312] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  878.186728] FASTCHG: request_mcu_irq: request_mcu_irq
[  878.227395] FASTCHG: fastcg_work_func:
[  878.377046] healthd: battery l=100 v=4380 t=29.5 h=2 st=2 c=-299 fc=2878000 cc=0 chg=a
[  878.412926] FASTCHG: dash_read: recv data:0x7f
[  878.423085] FASTCHG: request_mcu_irq: request_mcu_irq
[  878.423624] dashd: dash read
[  878.613952] QCOM-BATT: pl_fcc_vote_callback: total_fcc_ua=1950000
[  878.614019] QCOM-BATT: pl_fv_vote_callback: fv_uv=4400000
[  878.617035] FASTCHG: fastcg_work_func:
[  878.803793] FASTCHG: dash_read: recv data:0x52
[  878.812414] dashd: soc=100, temp=29, vbat=4381, ibat=-296
[  878.812515] SMBLIB: set_dash_charger_present: dash_present = 1, charger_present = 1
[  878.812529] FASTCHG: dash_dev_ioctl: REJECT_DATA
[  878.885807] FASTCHG: request_mcu_irq: request_mcu_irq
<repeats>
```