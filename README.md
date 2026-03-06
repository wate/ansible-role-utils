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
    task_from: prepare
```

### Finalize

```yaml
- name: Setup user home directories
  ansible.builtin.import_role:
    task_from: finalize
```

### Teardown

```yaml
- name: Cleanup unnecessary packages
  ansible.builtin.import_role:
    task_from: teardown
```

License
-----------------

Apache License 2.0
