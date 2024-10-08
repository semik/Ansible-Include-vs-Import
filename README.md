# Ansible include vs include_tasks

Deprecated `ansible.builtin.include` was passing a tag if used in playbook execution. The newly introduced `ansible.builtin.include_tasks` behaves differently, it includes a task but does not pass the tag used on the cmd line.

To demonstrate this, I've created a simple snipped task stored in file [include-import-demo-snippet.yml](include-import-demo-snippet.yml):
```yaml
- name: some task
  debug: msg="executed with {{ ansible_run_tags }}"
```

When used with code like this:
```yaml
    - name: Include
      ansible.builtin.include:
        file: include-import-demo-snippet.yml
      tags: [test_tag]
```
It is executed whether `--tags test_tag` is used or not.

But with newly propagated `ansible.builtin.include_tasks` and `--tags test_tag`:
```yaml
    - name: Include tasks without apply
      ansible.builtin.include_tasks:
        file: include-import-demo-snippet.yml
      tags: [test_tag]
```
The snippet is imported but because it is not tagged it is not executed. Moving related pieces into separate filles was the exact reason why I used to include in the first time, to reduce the amount of writing the same tag.

The same effect, passing the tag into all tasks in imported snipped has now to be written this way:
```yaml
    - name: Include tasks with apply
      ansible.builtin.include_tasks:
        file: include-import-demo-snippet.yml
        apply:
          tags: [test_tag]
      tags: [test_tag]
```

To see the difference compare the output of the following commands: 
```
ansible-playbook -i ./inventory.conf include-import-demo.yml
ansible-playbook -i ./inventory.conf include-import-demo.yml --tags test_tag
```
In the right panel, you should notice the missing output `executed with ['????']` after the first turquoise line declaring that the snipped was included. And that is the problem. Especially when you bulk replace `ansible.builtin.include` with `ansible.builtin.include_tasks` in your project to get rid of annoying warning... and do not test as I did not.

![image](https://github.com/user-attachments/assets/f2c1255a-deef-4018-a0ea-e0f721bfbe28)
