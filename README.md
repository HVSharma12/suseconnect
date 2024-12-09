# SUSEConnect

![Ansible Lint](https://github.com/HVSharma12/suseconnect/actions/workflows/ansible-lint.yml/badge.svg?branch=main)

This Ansible role is used to manage SUSE Linux system registrations with the SUSE Customer Center (SCC) or a local Subscription Management Tool (SMT) server. It automates the process of registering and deregistering systems, as well as managing additional products and modules on a SUSE system

This role includes:

- Registration of a SUSE system to SCC/SMT.
- Activation of specific add-on products or modules.
- Deregistration of systems or products.
- Removal of subscriptions.
- Precheck tasks to ensure a smooth registration process.

## Requirements

Before using this role, ensure that you have the following:

- A valid registration key for your SUSE products, which can be obtained with your SUSE subscription.
- For some products, additional registration keys are required.
- You need to know the internal product names for the products you're registering. These names can be found in [PRODUCTS.md](PRODUCTS.md).

## Role Variables

The following variables can be configured when using this role:

### suseconnect_base_product

- **Type**: dict
- **Description**: Defines the base product that should be activated on the target system. The dictionary should include the following keys:
  - `product`
    - **Type**: string
    - **Description**: Internal product name, see [PRODUCTS.md](PRODUCTS.md) for a list.
    - **Required**: No
  - `version`
    - **Type**: string
    - **Description**: Version of the product to be activated, defaults to the base OS version.
    - **Required**: No
  - `arch`
    - **Type**: string
    - **Description**: Architecture of the product, defaults to the OS architecture (`ansible_machine`).
    - **Required**: No
  - `key`
    - **Type**: string
    - **Description**: Registration key for the product.
    - **Required**: Yes
  - `email`
    - **Type**: string
    - **Description**: Email address used in SCC for registration.
    - **Required**: No

### suseconnect_subscriptions

- **Type**: list
- **Description**: List of additional modules or products to be registered on the target system.

  Each dictionary in the list can include the following keys:

  - `product`
    - **Type**: string
    - **Description**: Internal product name, see [PRODUCTS.md](PRODUCTS.md) for a list.
    - **Required**: Yes
  - `version`
    - **Type**: string
    - **Description**: Version of the product to be activated, defaults to the base OS version.
    - **Required**: No
  - `arch`
    - **Type**: string
    - **Description**: Architecture of the product, defaults to the OS architecture (`ansible_machine`).
    - **Required**: No
  - `key`
    - **Type**: string
    - **Description**: Additional registration key if required by the product.
    - **Required**: No
  - `email`
    - **Type**: string
    - **Description**: Email address used in SCC for registration.
    - **Required**: No

### suseconnect_reregister

- **Type**: bool
- **Description**: Whether to force re-registration of all products, regardless of current status (default: false).

### suseconnect_remove_subscriptions

- **Type**: bool
- **Description**: If set to `true`, this will remove the subscriptions that are mentioned in `suseconnect_subscriptions` (default: false).

### suseconnect_deregister

- **Type**: bool
- **Description**: Whether to deregister the system (default: false).

## Example Task

### Registering a SUSE Linux System

This example registers a SLES system and activates several modules:

```yaml
- name: Register with SCC and activate modules
  hosts: all
  become: true
  vars:
    suseconnect_base_product:
      product: '{{ ansible_distribution }}'
      key: '{{ sles_registration_key }}'

    suseconnect_subscriptions:
      - product: 'sle-module-basesystem'
      - product: 'sle-module-containers'
      - product: 'sle-module-server-applications'
      - product: 'sle-module-web-scripting'
      - product: 'PackageHub'

  tasks:
    - name: Register system and modules with SUSE Customer Center
      ansible.builtin.include_role:
        name: suseconnect
```

### Deregistering Products

This example shows how to deregister base products when they are no longer required on the system:

```yaml
- name: Deregister products from SCC
  hosts: all
  become: true
  vars:
    suseconnect_deregister: true

  tasks:
    - name: Deregister products from SUSE Customer Center
      ansible.builtin.include_role:
        name: suseconnect
```

### Removing Subscriptions

This task removes the subscriptions specified in the `suseconnect_subscriptions` variable by setting `suseconnect_remove_subscriptions` to `true`. Below is an example of removing subscriptions.

```yaml
- name: Clean up old subscriptions and add base subscription
  hosts: all
  become: true
  vars:
    suseconnect_remove_subscriptions: true  # Remove subscriptions

    suseconnect_subscriptions:
      - product: "sle-module-web-scripting"
      - product: "PackageHub"

  tasks:
    - name: Remove other subscriptions
      ansible.builtin.include_role:
        name: suseconnect
```

## License

This project is licensed under GPL-3.0.

## Author Information

### Modified and maintained by

- Harshvardhan Sharma
- Marcel Mamula

### Originally authored by

- Sebastian Meyer (<meyer@b1-systems.de>)  
  B1 Systems GmbH
