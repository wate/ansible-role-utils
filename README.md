utils
=================

Handles server utils tasks.

OS Platform
-----------------

- trixie
- bookworm

Example
-----------------

### Prepare

```yaml
- name: Update and upgrade packages
  ansible.builtin.import_role:
    name: utils
    task_from: package/update
```

### Finalize

```yaml
- name: Cleanup unnecessary packages
  ansible.builtin.import_role:
    name: utils
    task_from: package/autoremove
```

### Teardown

```yaml
- name: Create home directory symlinks
  ansible.builtin.import_role:
    name: utils
    task_from: filesystem/symlink
```

License
-----------------

Apache License 2.0
