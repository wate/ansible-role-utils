utils
=================

Handles server utils tasks.

OS Platform
-----------------

- trixie
- bookworm

Example
-----------------

```yaml
- name: Create symbolic links to the document root in the home directory
  ansible.builtin.import_role:
    name: utils
    task_from: filesystem/symlink
```

License
-----------------

Apache License 2.0
