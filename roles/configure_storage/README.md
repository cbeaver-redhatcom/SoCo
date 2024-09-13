# configure_storage

An Ansible role for managing lvm and gfs2 file systems for use in cluster deployment at Southern Company.

## Overview

This role calls three tasks files:

* `tasks/vgs.yml`
* `tasks/lvs.yml`
* `tasks/mkae_filesystems.yml`

### Role Variables

`defaults/main.yml`

```yaml
---
## volume group 
## used in tasks/lvs.yml, tasks/vgs.yml
vg_name_1: cldata_vg_01
vg_name_2: cldata_vg_02
vg_name_3: cldata_vg_03

## logical volume 
## used in tasks/lvs.yml
lv_name_1: home_oua_lv
lv_name_2: home_oma_lv
lv_name_3: home_flex_lv
lv_size: "100%FREE"

## gfs2 FS creation
## used in tasks/make_filesystems.yml
gfs_name_1: home_oua
gfs_name_2: home_oma
gfs_name_3: home_flex

## partition values
## used in tasks/partition.yml
LUN_1_partition_state: present
LUN_2_partition_state: present
LUN_3_partition_state: present

## LUNs for creating file systems
## used in tasks/partition.yml, tasks/set_vars_dynammic_LUNs.yml, tasks/set_vars_static_LUNs.yml, tasks/vgs.yml
__500GB_LUN: ""
__150GB_LUN_1: ""
__150GB_LUN_2: ""

## package names
## used in tasks/install_packages.yml
storage_packages: [lvm2,watchdog,lvm2-lockd,gfs2-utils,dlm]
```

### tasks/vgs.yml

After LUNs are defined for use, volume groups are created, and locks are started for each volume group.  Ideally, we use the lvg module, but we were getting errors associated with the LUNs each VG was using.  So, we pivoted to using command module with a conditional.

```yaml
--- 
- name: Grab vgs output
  command:
    cmd: vgs
  register: vgs_info
  changed_when: false 

- name: Create "{{ vg_name_1 }}"
  command:
    cmd: "vgcreate --shared {{ vg_name_1 }} /dev/mapper/{{ __500GB_LUN.split('-')[-1] }}"
  when: not vgs_info is search(vg_name_1) # run when VG name not in vgs output

- name: Check on lock status of "{{ vg_name_1 }}"  ## See EOF for lvmlockctl --info output example
  command: 
    cmd: lvmlockctl --info
  register: lock_info_1

- name: Start DLM ("{{ vg_name_1 }}")
  command:
    cmd: vgchange --lock-start "{{ vg_name_1 }}"
  when: not lock_info_1 is search(vg_name_1) 

- name: Create cldata_vg_02
  command:
    cmd: "vgcreate --shared {{ vg_name_2 }} /dev/mapper/{{ __150GB_LUN_1.split('-')[-1] }}"
  when: not vgs_info is search(vg_name_2) # run when VG name not in vgs output

- name: Check on lock status of cldata_vg_02
  command: 
    cmd: lvmlockctl --info
  register: lock_info_2

- name: Start DLM (cldata_vg_02)
  command:
    cmd: vgchange --lock-start "{{ vg_name_2 }}"
  when: not lock_info_2 is search(vg_name_2)

- name: Create cldata_vg_03
  lvg:
  command:
    cmd: "vgcreate --shared {{ vg_name_3 }} /dev/mapper/{{ __150GB_LUN_2.split('-')[-1] }}"
  when: not vgs_info is search(vg_name_3) # run when VG name not in vgs output

- name: Check on lock status of cldata_vg_03
  command: 
    cmd: lvmlockctl --info
  register: lock_info_3

- name: Start DLM (cldata_vg_03)
  command:
    cmd: vgchange --lock-start "{{ vg_name_3 }}"
  when: not lock_info_3 is search(vg_name_3)

## lvg module was complaining about the multipath device, so we need to use command module in this file.
```
### tasks/lvs.yml

NOTE: Module `community.general.lvol` is used for logical volume creation.  Due to locked down environment, community.general collection could not be installed per usual process.  The community.general collection had to be uploaded via ssh from employee workstation within Southern Company.

This task file is very straightforward.  It creates the three logical volumes.  There's a 1:1 mapping between volume groups previously created in `tasks/vgs.yml` and logical volumes created in `tasks/lvs.yml`.  

```yaml
---
- name: Create LV home_out_lv
  community.general.lvol:
    vg: "{{ vg_name_1 }}"
    lv: "{{ lv_name_1 }}"
    size: "{{ lv_size }}"
    active: yes
    state: present

- name: Create LV home_oma_lv
  community.general.lvol:
    vg: "{{ vg_name_2 }}"
    lv: "{{ lv_name_2 }}"
    size: "{{ lv_size }}"
    active: yes
    state: present

- name: Create LV home_flex_lv
  community.general.lvol:
    vg: "{{ vg_name_3 }}"
    lv: "{{ lv_name_3 }}"
    size: "{{ lv_size }}"
    active: yes
    state: present
```

### tasks/make_filesystems.yml

This task file is also straightforward.  We grab the output of `mount` and ensure the target filesystems don't exist before creating them.  We use `command`, since the `file` module lacks necessary parameters for gfs2 file system creation.  Tasks leverage variables found in `defaults/main.yml`.

```yaml
---
- name: Grab mount output
  command:
    cmd: mount
  register: mount_out

- name: Create filesystem -- "{{ gfs_name_1 }}"
  command:
    cmd: "mkfs.gfs2 -j2 -p lock_dlm -t {{ cluster_name }}:{{ gfs_name_1 }} /dev/cldata_vg_01/{{ gfs_name_1 }}_lv"
    stdin: y
  when: not mount_out is search(gfs_name_1)  # runs when fs to be created not found in mount output

- name: Create filesystem -- "{{ gfs_name_2 }}"
  command:
    cmd: "mkfs.gfs2 -j2 -p lock_dlm -t {{ cluster_name }}:{{ gfs_name_2 }} /dev/cldata_vg_02/{{ gfs_name_2 }}_lv"
    stdin: y
  when: not mount_out is search(gfs_name_2) # runs when fs to be created not found in mount output

- name: Create filesystem -- "{{ gfs_name_3 }}"
  command:
    cmd: "mkfs.gfs2 -j2 -p lock_dlm -t {{ cluster_name }}:{{ gfs_name_3 }} /dev/cldata_vg_03/{{ gfs_name_3 }}_lv"
    stdin: y
  when: not mount_out is search(gfs_name_3) # runs when fs to be created not found in mount output
```

## License

MIT

## Author Information

Cory Beaver
