# select_LUN

An Ansible role for managing statically or dynamically selected storage devices for use in cluster deployment at Southern Company.

## Overview

This role relies on the following variables set when running cluster deployment playbooks:

* `__500GB_LUN`
* `__150GB_LUN_1`
* `__150GB_LUN_2`

Three tasks are executed in `tasks/main.yml`:

```yaml
---
- name: Fail play if 150GB LUNs are defined statically and dynamically
  fail:
    msg: "LUNs must both be either statically defined or dynamically defined."
  when: >
    ( __150GB_LUN_1 != "" and __150GB_LUN_2 == "") or
    ( __150GB_LUN_1 == "" and __150GB_LUN_2 != "")

- name: Check and prepare role variables (static LUNs)
  include_tasks: set_vars_static_LUNs.yml
  when:
    - __150GB_LUN_1 != ""
    - __150GB_LUN_2 != ""

- name: Check and prepare role variables (dynamic LUNs)
  include_tasks: set_vars_dynamic_LUNs.yml
  when: 
    - __150GB_LUN_1 == ""
    - __150GB_LUN_2 == ""
```

Firstly, if one 150GB LUN is defined statically, and the other not, the play fails automatically.  If both 150GB LUNs are defined, `set_vars_static_LUNs.yml` runs.  If 150GB LUN values are left blank, the code will initiate `set_vars_dynamic_LUNs.yml` and select LUNs dynamically based in certain parameters.  The conditionals found in the code rely solely on `ansible_facts.devices` dict.

## Role Variables

Defined in `vars/main.yml`

`__150GB_LUN_list_length:`

int, default: `2`

Expected number of 150GB LUNs to be used in cluster deployment.

`total_LUNs_list_length:`

int, default: `3`

Expected number of total LUNs to be used in cluster deployment.

### LUN Parameters

500GB LUN:

* Size is 500GB
* Holders length is 0 (holders represent devices the LUN is providing storage to)
* There are no partitions on the device

150GB LUN:

* Size is 150GB
* Holders length is 0 (holders represent devices the LUN is providing storage to)
* There are no partitions on the device

## Fail-safes

The following fail-safes are in place to cause the code to fail before any storage device is mistakenly used:

* Expected number of 150GB LUNs not found

```yaml
- name: Fail of length of __150GB_LUN_list != "{{ __150GB_LUN_list_length }}"
  fail:
    msg: "Number of 150GB LUNs to configure filesystems is not {{ __150GB_LUN_list_length }}"
  when: __150GB_LUN_list | unique | length != __150GB_LUN_list_length | int
```

* Statically defined __500GB_LUN does not meet requirements

```yaml
- name: Check that statically defined __500GB_LUN value meets requirements
  fail:
    msg: "__500GB_LUN value did not meet requirements: [500GB in size,no holders,no partitions] -- Review provided value for __500GB_LUN ." 
  loop: "{{ lookup('ansible.builtin.dict', ansible_facts.devices) }}"
  when: 
    - item.value.links.ids[0] is defined
    - item.value.links.ids[0] == __500GB_LUN  
    - item.value.size != "500.00 GB" or item.value.holders | length > 0 or item.value.partitions != {}
    - _500GB_LUN != ""
```

* __500GB_LUN could not be dynamically defined

```yaml
- name: Fail play if _500GB_LUN could not be dynamically defined
  fail: 
    msg: "No LUNs could be assigned to __500GB_LUN due to not meeting requirements"
  when: __500GB_LUN == ""
```

* Total number of LUNs to be defined does not match total amount of LUNs defined.

```yaml
- fail:
    msg: "Required amount of acceptable LUNs not found.  Expected {{ total_LUNs_list_length }}.  Found {{ LUN_list | unique | length"}}.
  when: LUN_list | unique | length != total_LUNs_list_length | int
```
## License

MIT

## Author Information

Cory Beaver
