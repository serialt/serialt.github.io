+++
title = 'Python ansible module'
date = 2023-11-22T21:22:27+08:00
draft = false

tags = ["python-ansible","ansible-module"]
categories = ["DevOps"]

+++

# Ansible Module

参考链接：https://ansible.leops.cn/dev/modules/

1、自定义模块开发

该模块的目的是在远程主机上将远程源文件复制到远程目标文件

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
# Copyright: (c) 2020, lework
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)

ANSIBLE_METADATA = {'metadata_version': '1.0',
            'status': ['preview'],
                'supported_by': 'community'}

DOCUMENTATION = '''
---
module: remote_copy
short_description: Copy a file on the remote host
version_added: "2.9"
description:
  - The remote_copy module copies a file on the remote host from a given source to a provided destination.
options:
  source:
    description:
      - Path to a file on the source file on the remote host
    required: true
  dest:
    description:
      - Path to the destination on the remote host for the copy
    required: true
author:
  - "Lework"
'''

EXAMPLES = '''
# Example from Ansible Playbooks
- name: backup a config file
  remote_copy:
    source: /tmp/foo
    dest: /tmp/bar
'''

RETURN = '''
source:
    description: Path to a file on the source file on the remote host
    type: str
    returned: success
    sample: "/path/to/file.name"
dest:
    description: Path to the destination on the remote host for the copy
    type: string
    returned: success
    sample: "/path/to/destination.file"
'''

import os
import shutil
from ansible.module_utils.basic import AnsibleModule

def main():
    module_args = dict(
            source=dict(required=True, type='str'),
            dest=dict(required=True, type='str')
    )

    result = dict(
        changed=False,
        source='',
        dest=''
    )

    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    if module.check_mode:
       module.exit_json(**result)

    if not os.path.isfile(module.params['source']):
        module.fail_json(msg='The '+ module.params['source'] +' file was not found', **result)

    try:
        shutil.copy(module.params['source'], module.params['dest'])
    except Exception as e:
        module.fail_json(msg=e, **result)

    result['source'] = module.params['source']
    result['dest'] = module.params['dest']

    if os.path.isfile(module.params['dest']):
        result['changed'] = True

    remote_facts = {'rc_source': module.params['source'], 'rc_dest': module.params['dest'] }
    result['ansible_facts'] = remote_facts

    module.exit_json(**result)

if __name__ == '__main__':
    main()
```

