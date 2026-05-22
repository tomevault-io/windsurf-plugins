---
trigger: always_on
description: A package is a special atopile project that is intended to be shared and reused in other designs. A package is denoted by a `package` section in the ato.yaml file. A package aims to design a basic implementation of a given integrated circuit component. The requirement is for this package to include all the necessary components to get the core device working, such as decoupling capacitors, pullup/pulldown resistors, and more. Most importantly, since this package will be reused in a different desi
---


# 0 Context
A package is a special atopile project that is intended to be shared and reused in other designs. A package is denoted by a `package` section in the ato.yaml file. A package aims to design a basic implementation of a given integrated circuit component. The requirement is for this package to include all the necessary components to get the core device working, such as decoupling capacitors, pullup/pulldown resistors, and more. Most importantly, since this package will be reused in a different design, it is critical to correctly create, expose, and connect all the external interfaces properly. External interfaces should be created using the proper object type (ElectricPower/ElectricLogic/etc.) and clearly defined with docstrings.
Some examples of critical interfaces to expose for users include but are not limited to:
  - ElectricPower for the power rails - `add electricpower.required = true` for required power rails
  - SPI/I2C/I2S/etc. - communication interfaces should be defined and connected to the pins of the physical package
  - EnablePins - `enable_pin.line.required = true` so users don't ignore or forget to connect the enable line somewhere

The purpose of the usage build is to show users how to use the package. This means this example should show the preferred method of use, and is most helpful if it shows the user how to use many of the interfaces.

# 1 Building a new package

## 1.1 File Structure

```
packages/
  packages/
    <package_name>/
        layouts/
        parts/
        ato.yaml
        <package_name>.ato
        README.md
        usage.ato
```

## 1.2 Steps to create a new package

1. Create a new directory `<package_name>` as stated in the file structure.
2. Create ato.yaml, `<package_name>.ato`, README.md, usage.ato
3. Look through other packages for inspiration
4. Create part using tool call 'search_and_install_jlcpcb_part'
   4.1 Inspect the part ato file in the parts/ directory
5. Import the part into the main ato file
6. Read the datasheet for the device
7. Populate the files with the correct information (see below)
   7.1 Create interfaces and connect them
   7.2 Add decoupling caps where needed
   7.3 Add i2c addressor if device has configurable address
   If format is: <n x fixed address bits><m x pin configured address bits>
   use addressor module:

- Use `Addressor<address_bits=N>` where **N = number of address pins**.
- Connect each `address_lines[i].line` to the corresponding pin, and its `.reference` to a local power rail.
- Set `addressor.base` to the lowest possible address and `assert addressor.address is i2c.address`.

  7.4 Other configuration etc

8. Review the content wholistically again
9. Build the main build target using CLI `ato build`
10. Make sure to make use of the LSP and build errors

## 1.3 Additional Notes & Gotchas (generic)

- Multi-rail devices (VDD / VDDIO, AVDD / DVDD, etc.)

  - Model separate `ElectricPower` interfaces for each rail (e.g. `power_core`, `power_io`).
  - Mark each `.required = True` if the device cannot function without it, and add voltage assertions per datasheet.

- Using the right type of signal: Eletrical, ElectricLogic, ElectricSignal, DifferentialPair, I2C, SPI, ...

  - Electrical: Represents a basic electrical object, does not have a voltage reference.
  - ElectricSignal: Represents an electric signal with a voltage domain reference (electricsignla.reference) which should be connected to the appropriate ElectricPower object. The electricsignal.line can represent any voltage between the hv and lv of the reference. Useful for things like analog signals, voltage divider outputs, etc.
  - ElectricLogic: Represents an electric logic, which is a special type of electric signal that should only take the discrete values of reference.hv and reference.lv. electriclogic.line  will often be soft pulled up or down to its reference rails.
  - DifferentialPair: Represents a pair of ElectricLogics that share a reference ground (still need to connect the reference ElectricPower to something) and carry a signal differentially between the p.line and n.line
  - I2C, SPI, etc: Represent interfaces of common communication protocols. Investigate the files in the standard library to find which signals are available for each interface. Interfaces should be used wherever possible as a layer of abstraction. Instead of connecting sensor.sda_pin~micro.sda_pin and sensor.scl_pin~micro.scl_pin, I2C interfaces should be described in the definition of the sensor and the micro such that at application level they can be connected via sensor.i2c ~ micro.i2c

- Use arrays for multiple channels or repetitve signals/modules
  e.g `vouts = new ElectricLogic[4]` and access them with for-loops
  e.g `gpios = new ElectricLogic[10]` and access them with a for-loop such as
      for gpio in gpios:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atopile/packages](https://github.com/atopile/packages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
